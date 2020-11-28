---
layout: post
title:  "Playing with prime numbers and sieve of Eratosthenes"
date:   2020-11-23 10:56:33 -0400
categories: [C++ programming language, math]
tags: ["C++", prime number, sieve of Eratosthenese]
toc: true
---

This post is motivated by [the Euler project's third problem](https://projecteuler.net/problem=3), asking to find the largest prime factor of the number `n=600,851,475,143`. In my humble opinion, the most challenging issue of the problem is how to efficiently determine whether a number is a prime number, and the best candidate is [the sieve of Eratosthenes algorithm](https://en.wikipedia.org/wiki/Sieve_of_Eratosthenes). In other words, my strategy is to generate all prime numbers up to the given `n` to ensure a fixed time (i.e., `O(1)`) for prime-checking. This post mainly focuses only on the sieve of Eratosthenes algorithm, C++ implementations, and some optimizations so that I can list all prime numbers up to the given `n`.

## The sieve of Eratosthenes
The sieve of Eratosthenes is an ancient algorithm to find all the prime numbers less than or equal to a given integer `n`. The detail of the algorithms is as follows:

1. Create an array of consecutive integers from `2` to `n`.
1. Take the first element of the array (i.e., `2`) and mark all other elements which are multiples of the element (except itself) as `nonprime`.
1. Repeat step 2 for the next element that has not been marked until reaching `n`.

## Implementations

**Version 1**

In this version, I simply implement the algorithm with a minor optimization. Instead of using an array of consecutive integers, I use an array of boolean variables to indicate whether a number is prime. That is, if `primes[i] == true` , then `i` is a prime number. The detail of of the implementation is as follows:

```C++
#include <algorithm>
#include <memory>

unique_ptr<bool[]> create_SoE_v1 (const size_t &n) {
    unique_ptr<bool[]> primes{ make_unique<bool[]>(n+1)};
    fill_n(primes.get(), n+1, true);
    // 0 and 1 are not considered as prime numbers
    primes[0] = false;
    primes[1] = false;

    // Mark non-prime numbers
    for(size_t i = 3; i <= n; i++) {
        if(primes[i] == true) {
            for(size_t j = 2*i; j <= n; j += i) {
                primes[j] = false;
            }
        }
    }

    return move(primes);
}
```

As denoted in the following table, the algorithm can find about `455` million prime numbers between `2` and `10` billion in about 2 mins when running on my Dell Precision 5820 PC (Intel(R) Xeon(R) 3.20GHz CPU and 32GB of RAM). This is quite good. However, I got a memory error when setting `n` to 100 billion.

<table>
  <caption style="caption-side:bottom">Evaluation of create_SoE_v1 </caption>
  <tr>
    <th>n</th>
    <th>\# of prime numbers</th>
    <th>Time (s)</th>
  </tr>
  <tr>
    <td>10,000,000</td>
    <td>664,579</td>
    <td>0.82</td>
  </tr>
  <tr>
    <td>100,000,000</td>
    <td>5,761,455</td>
    <td>8.59</td>
  </tr>
  <tr>
    <td>1000,000,000</td>
    <td>50,847,534</td>
    <td>93.31</td>
  </tr>
  <tr>
    <td>10,000,000,000</td>
    <td>455,052,511</td>
    <td>130.19</td>
  </tr>
  <tr>
    <td>100,000,000,000</td>
    <td colspan="2" style="text-align:center;">memory error</td>
  </tr>
</table>

**Version 2**

Observe that:
- If `n` is not a prime number, I can always find a pair of integers `p` and `q` such that both `p` and `q` are less than `n`, and `p` is a prime number.
- If `n` is a prime number, the algorithm ensures that all nonsize_t-prime numbers smaller that `n` are marked.
Thus, instead of checking all the way up to `n`, I need to check only up to the square root of `n`.

```C++
unique_ptr<bool[]> create_SoE_v2(const size_t &n) {
    unique_ptr<bool[]> primes{ make_unique<bool[]>(n+1)};
    fill_n(primes.get(), n+1, true);
    // 0 and 1 are not considered as prime numbers
    primes[0] = false;
    primes[1] = false;

    // Mark non-prime numbers
    size_t sqrt_n = static_cast<size_t>(sqrt(n));
    for (size_t i = 3; i <= sqrt_n; i += 2) {
        if(primes[i] == true) {
            for(size_t j = 2*i; j <= n; j += i) {
                primes[j] = false;
            }
        }
    }

    return move(primes);
}
```

This version is approximately one third faster than the first one, as denoted in Figure 1. However, it still produces the memory error when `n` is `100` billion.

![Figure 1: Time comparison 1](https://github.com/vuanhduy/vuanhduy.github.io/blob/master/_posts/images/soe_time_comparison_1.png)


**Version 3**

In this version, I aim to optimize the memory by replacing the boolean variables with bits. A simple approach is to utilize the built-in [`bitset` class](https://en.cppreference.com/w/cpp/utility/bitset), but the set's length must be provided at compiling time, which I do not like.  So, I develop a new solution using an array of `std::byte`'s. That is, I will divide the numbers into groups of 8 elements and have each bit of the corresponding `std::byte` indicating whether an element of the group is prime or not. Note that when `n` is not divisible by `8`, the last group is not full, so we must clean the trailing bits.

Additionally, except `2`, there is no other prime number that is even. Thus, I can ignore all even numbers and have the array start from `3`. This optimization helps to reduce not only  half of the required memory but also the number of loops. Without further ado, here is the major part of the third implementation:

```C++
SoE(const size_t &n): m_upper_bound(n){
    size_t number_of_blocks = (((n >> 1) - 1) >> 3) + 1;
    m_primes = make_unique<byte[]>(number_of_blocks);
    fill_n(m_primes.get(), number_of_blocks, static_cast<byte>(0xFF));

    // a lambda to check whether number at index is prime
    auto check = [&](const size_t &index) {
        size_t block_index = ((index >> 1) - 1) >> 3;
        size_t offset = ((index >> 1) - 1) & 0x7;
        byte mask{0x80};
        mask >>= offset;
        return to_integer<int>(m_primes[block_index] & mask) > 0;
    };

    // a lambda to clear the bit at index
    auto mark = [&](const size_t &index) {
        size_t block_index = ((index >> 1) - 1) >> 3;
        size_t offset = ((index >> 1) - 1) & 0x7;
        byte mask{0x80};
        mask >>= offset;
        m_primes[block_index] &= ~mask;
    };

    // Clear trailing bits
    size_t last_group_size = ((n >> 1) - 1) & 0x7;
    byte mask{0xFF};
    mask >>= last_group_size;
    m_primes[number_of_blocks - 1] ^= mask;

    // Mark non-prime numbers
    size_t sqrt_n = static_cast<size_t>(sqrt(n));
    for (size_t i = 3; i <= sqrt_n; i += 2) {
        if (check(i)) {
            for (size_t j = 3 * i; j <= n; j += 2*i) {
                mark(j);
            }
        }
    }
};
```
As you may notice, I also had some minor optimization by using bitwise operators instead of division or modulo.

The time comparison between the three versions is shown in the following figure, and the last version, of course,  outperforms the other two. Additionally, this is also the only one that can find all prime numbers between 2 and 200 billion, which is still quite far from my goal.

![Figure 2: Time comparison 2](https://github.com/vuanhduy/vuanhduy.github.io/blob/master/_posts/images/soe_time_comparison_2.png)

At this point, I am thinking of parallelizing the algorithm. That is, I will divide the range into smaller segments and process them in parallel. However, the latter segments likely have less prime numbers than the former ones making threads processing latter segments have to invoke the `mark` lambda more often than those of former segments. In other words, the challenge of parallelizing the algorithm is how to balance the workload between threads. Thus, I would like to have these kinds of optimization in a separate post.


## Conclusion

This post presented the accent algorithm named Sieve of Eratosthenes and 3 different implementations in C++, optimizing running time and storing space. Although the original goal is to find all prime numbers between `2` and `600,851,475,143` to solve the Euler project's third problem, the proposed implementation can only do `[2, 200,000,000,000]`. Another possible optimization is to divide the range into smaller segments and process them in parallel, which I will present in another post soon.

Finally, the source code of the implementations can be found [here.](https://github.com/vuanhduy/PrimeNumbersAndSieves)

## External links:
1. [Euler project](https://projecteuler.net/)
1. [Wikipedia page of sieve of Eratosthenes](https://en.wikipedia.org/wiki/Sieve_of_Eratosthenes)
1. [Wikipedia page of sieve of Sundarm](https://en.wikipedia.org/wiki/Sieve_of_Sundaram)
1. [Wikipedia page of sieve of Atkin](https://en.wikipedia.org/wiki/Sieve_of_Atkin)
