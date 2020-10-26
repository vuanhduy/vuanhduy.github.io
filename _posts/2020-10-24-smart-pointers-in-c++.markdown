---
layout: post
title:  "C++ smart pointers"
date:   2020-10-24 10:56:33 -0400
categories: [C++, smart pointers]
---

A **smart pointer** is a class that manages a dynamically allocated object,ensuring the dynamically allocated object is properly cleaned up at the appropriate time (usually when the smart pointer goes out of scope).

C++11 standard library ships with 4 smart pointer classes: `std::auto_ptr` (which you shouldn’t use -- it’s being removed in C++17), `std::unique_ptr` `std::shared_ptr`, and `std::weak_ptr`.

# Smart pointer types
## `std::unique_ptr`

The `std::unique_ptr` should be used to manage a single dynamically-allocated object. That is, there must be only a single pointer pointing to the dynamically-allocated object. Internally, as the `std::unique_ptr` variable is allocated on the stack, it’s guaranteed to eventually go out of scope, and when it does, the system will delete the heap-allocated content that the variable is managing.

The `std::unique_ptr` has an overloaded `*` and `->` operations that return the **reference** and **pointer** to the content being managed, respectively.

Because the `std::unique_ptr` is designed with move semantics in mind, copy initialization and copy assignment are disabled. If you want to transfer the contents managed by `std::unique_ptr`, you must use move semantics (e.g., via `std::move`) (See Example 2.)

`std::make_unique` is a templated function creating a new `std::unique_ptr` pointer to an object of the template type. The newly created object is initialized with the arguments passed into the function. Furthermore the `std::make_unique` can resolve an exception safety issue that can result from `C++` leaving the order of evaluation for function arguments unspecified (see [here](#markdown-header-the-exception-safety-issue) for the details).

On returning a `std::unique_ptr` pointer from an function, if this value is not assigned to anything, the temporary return value will go out of scope and be cleaned up. If it is assigned, in C++14 or earlier, move semantics will be employed to transfer the temporary value from the return value to the object assigned to, and in C++17 or newer, the return will be elided. This makes returning a resource by `std::unique_ptr` much safer than returning raw pointers! (see Example 4.)

On passing `std::unique_ptr` to a function, the function will take ownership of the contents of the pointer.  Note that because copy semantics have been disabled for `std::unique_ptr`, you’ll need to use `std::move` to actually pass the variable in (see Example 5.)

To pass a `std::unique_ptr` by reference, you can use the `get()` member function


Some examples:
```C++
#include <iostream>
#include <utility> // for std::move
#include <memory> // for smart pointers

std::unique_ptr<MyClass> create_my_object();
void pass_by_value(std::unique_ptr<MyClass> my_object);
void pass_by_reference(MyClass *my_object);

int main(int argc, const char * argv[]) {
    // ==================== Example 1 ====================
    // Create a new smart pointer points to an object of MyClass
    std::unique_ptr<MyClass> my_object{ new MyClass() };

    // Create another smart pointer starts as nullptr
    std::unique_ptr<MyClass> my_object2{};

    // ==================== Example 2 ====================
    // Ownership
    // NOTE: res2 = res1; Won't compile becuase copy assignment is disabled
    my_object2 = std::move(my_object); // my_object2 assumes ownership while my_object is now set to nullptr

    // ==================== Example 3 ====================
    // Using make_unique to create a single dynamically allocated MyClass object
    auto my_object3{ std::make_unique<MyClass>(/*parameters of MyClass' constructor*/) };
    // Using make_unique to create a dynamically allocated array of MyClass objects of length 4
    auto my_object_array{ std::make_unique<MyClass[]>(4) };

    // ==================== Examp 4 ====================
    // The smart pointer returned from create_my_object will be "moved" to ptr
    auto ptr{ create_my_object() };

    // ==================== Example 5 ====================
    pass_by_value(std::move(my_object2)); // After this call, my_object2 is set to nullptr.

    // ==================== Example 6 ====================
    pass_by_reference(my_object3.get());  // After this call, my_object3 still points to the same object as before the call.
    return 0;
}

// A function that returns a smart pointer. 
std::unique_ptr<Resource> create_my_object();
{
    return std::make_unique<MyClass>();
}

void pass_by_value(std::unique_ptr<MyClass> my_object) {
    // my_object will be destructed at the end of this function
}

void pass_by_reference(MyClass *my_object) {
    // Any changes of my_object inside this function can be obtained by the caller.
}
```

## `std::shared_ptr`

`std::shared_ptr` is a means to solve the case where you need multiple smart pointers co-owning a resource. Internally, `std::shared_ptr` keeps track of how many `std::shared_ptr` are sharing the resource. As long as at least one `std::shared_ptr` is pointing to the resource, the resource will not be deallocated, even if individual `std::shared_ptr` are destroyed. As soon as the last `std::shared_ptr` managing the resource goes out of scope (or is reassigned to point at something else), the resource will be deallocated (see Example 7.)

Note that if an object is used to initialize two or more `std::shared_ptr` pointers, these pointers are indepent, aren't aware of each other, and will cause a crash at some point  (see Example 8.) Internally, each `std::shared_ptr` uses two pointers: one points to the object being managed, and the other points to a __control block__, which is a dynamically allocated object that tracks of a bunch of stuff, including how many `std::shared_ptr` are pointing at the object. When we create multiple independent `std::shared_ptr` pointing to the same object, each `std::shared_ptr` will have one pointer pointing to the object and another one pointing to its own control block indicating that it is the only pointer owning that object. Thus, when one of the pointers goes out of scope, it thinks that it's the only owner of the object and deallocates the object, not realizeing that there are other pointers managing the object.

`std::make_shared()`, which is similar to `std::make_unique`, is a templated function that and should be used to make a `std::shared_ptr` (see Example 9). The `std::make_shared()` does not only solve exception safety issue but also is faster than the `std::shared_ptr` constructor. Recall that each `std::shared_ptr` uses two pointers internally: one points to the resource being managed, and the other points to a control block. When a `std::shared_ptr` is created via a `std::shared_ptr` constructor, the memory for the managed object (which is usually passed in) and control block (which the constructor creates) are allocated separately. However, when using `std::make_shared()`, this can be optimized into a single memory allocation, leading to better performance.

A `std::unique_ptr` can be converted into a `std::shared_ptr` via a special `std::shared_ptr` constructor that accepts a `std::unique_ptr` r-value. However, `std::shared_ptr` can not be safely converted to a `std::unique_ptr`. Thus, if you’re creating a function that is going to return a smart pointer, you’re better off returning a `std::unique_ptr` and assigning it to a `std::shared_ptr` if and when that’s appropriate.


Some examples:
```C++
#include <iostream>
#include <memory> // for smart pointers

int main(int argc, const char * argv[]) {
    // ==================== Example 7 ====================
	  MyClass *my_object = new MyClass();
    // Have the my_object owned by std::shared_ptr
    {
        std::shared_ptr<MyClass> ptr1(my_object);
        // This should show: ptr1.use_count: 1
        std::cout << "ptr1.use_count: " << ptr1.use_count() << std::endl;
        {
            // NOTE: ptr1 is used to initilized the 2nd pointer.
            std::shared_ptr<MyClass> ptr2(ptr1);
            // This should show: ptr1.use_count: 2
            std::cout << "ptr1.use_count: " << ptr1.use_count() << std::endl;
        } // ptr2 goes out of scope here, decreasing the number of refrences to my_object to 1. As there is still a reference to my_object, it won't be deallocated.
    } // ptr1 goes out of scope here decreasing the number of reference to my_object to 0, so my_object could be deallocated from here.

    // ==================== Example 8 ====================
    // The below codes will cause a crash
    // {
    //     MyClass *my_object2 = new MyClass();
    //     std::shared_ptr<MyClass> ptr1(my_object2);
    //     std::shared_ptr<MyClass> ptr2(my_object2);
    // }

    // ==================== Example 9 ====================
    auto ptr1 = std::make_shared<MyClass>();

    return 0;
}
```

## `std::weak_ptr`

`std::weak_ptr` is designed to solve the **circular dependency issue** (see [here](#markdown-header-the-circular-dependency-issue) for the details). A `std::weak_ptr` works as an observer, which can observe and access the same object as a `std::shared_ptr` (or other `std::weak_ptrs`) but is not considered an owner. Recall that when a `std::shared_ptr` goes out of scope,it only considers whether other `std::shared_ptr` are co-owning the object (`std::weak_ptr` does not count!)

`std::weak_ptr` are not directly usable (i.e., they have no operator `->`.) To use it, the `std::weak_ptr` must be converted into a `std::shared_ptr` using the `lock()` member function (see Exampl 11).

Some examples:
```C++
#include <iostream>
#include <memory> // for smart pointers

class MyClass
{
...
public:
    std::weak_ptr<MyClass> m_ptr; // initially created empty
...
};

int main(int argc, const char * argv[]) {

    // ==================== Example 10 ====================
    // using weak_ptr to break the cycle references
    auto my_object1 = std::make_shared<MyClass>();
	  auto my_object2 = std::make_shared<MyClass>();

    my_object1->m_ptr = my_object2;
		my_object2->m_ptr = my_object1;

    // ==================== Example 11 ====================
    auto my_object3 = my_object1.m_ptr.lock()
    // my_object3 can be used as a shared_ptr.

    return 0;
}
```

# Mics
## The exception safety issue
Consider an expression like this one:

```C++
some_function(std::unique_ptr<T>(new T), function_that_can_throw_exception());
```
The compiler is given a lot of flexibility in terms of how it handles this call. It could create a new `T`, then call `function_that_can_throw_exception()`, then create the `std::unique_ptr` that manages the dynamically allocated `T`. If `function_that_can_throw_exception()` throws an exception, then the `T` that was allocated will not be deallocated, because the smart pointer to do the deallocation hasn’t been created yet. This leads to `T` being leaked.

`std::make_unique()` (resp. `std::make_shared()`) doesn’t suffer from this problem because the creation of the object T and the creation of the `std::unique_ptr` (resp. `std::shared_ptr`) happen inside the `std::make_unique()` (resp. `std::make_shared()`) function, where there’s no ambiguity about order of execution.

## The circular dependency issue
A **Circular dependency** (also called a **circular reference**, **cyclical reference**,  or a **cycle**) is a series of references where each object references the next, and the last object references back to the first, causing a referential loop (see Example 10). Note that,
- In the context of shared pointers, the references will be pointers.
- This cyclical reference issue can even happen with a single object if it has a `std::shared_ptr` member referencing to the object itself (see Example 11)

```C++
#include <iostream>
#include <memory> // for smart pointers

class MyClass
{
public:
    std::shared_ptr<MyClass> m_ptr; // initially created empty
};

int main(int argc, const char * argv[]) {
    // ==================== Example XX ====================
    auto my_object1 = std::make_shared<MyClass>();
	  auto my_object2 = std::make_shared<MyClass>();

    my_object1->m_ptr = my_object2;
		my_object2->m_ptr = my_object1;

    // ==================== Example XX ====================
    auto my_object3 = std::make_shared<MyClass>();
    my_object3->m_ptr = my_object3;

    return 0;
}
```
