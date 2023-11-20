---
layout: post
title:"Design pattern: Single Responsibility Principle"
date: 2023-11-16 10:56:33 -0400
categories: [C++ programming language, Design Pattern]
tags: ["`C++`", ", Design Pattern", "SOLID"]
toc: true
---

As software developers, we often grapple with the challenge of writing clean, maintainable and flexible code. One key principle aiding us in achieving these goals is the Single Responsibility Principle (SRP), one of the SOLID principles of object-oriented design. In this blog post, we will delve into the essence of SRP, exploring its significance through a practical example in C++.

## What is the Single Responsibility Principle?
At its core, the Single Responsibility Principle advocates that a class should be designed with a single responsibility and should not take on additional responsibilities. A class's primary responsibility should be clearly defined and focused. Just as a programmer's primary responsibility is to code.


## Realizing SRP in C++
Let's consider a scenario where we are developing a `Rectangle` class in C++. Initially, the class manages a rectangle's height and width:

```C++
#include <iostream>
#include <vector>
#include <string>
#include <fstream>

class Rectangle {
public:
    Rectangle(int height, int width) : mHeight(height), mWidth(width) {}

    void setHeight(int height) {
        mHeight = height;
    }

    void setWidth(int width) {
        mWidth = width;
    }

    int getHeight() const {
        return mHeight;
    }

    int getWidth() const {
        return mWidth;
    }

private:
    int mHeight;
    int mWidth;
};
```


While this implementation appears sound, issues arise when the need for display functionality arises - rendering a rectangel to a screen. A common mistake is to extend the `Rectangle` class with display-related methods:

```C++
class Rectangle {
public:
    // Existing methods...

    void draw() const {
        // Draw the rectangle ...
    }

    // Other display-related methods...
};
```


Althought this seems convenient, it violates the Single Responsibility Principle: the `Rectangle` class is now responsible not only for managing rectangle attributes but also for display operations. This violation can lead to maintenance challenges as the codebase expands.


To adhere to SRP, we must separate concerns. In this case, we introduce a `Canvas` class responsible for drawing operations:

```C++
class Canvas {
public:
    static void draw(const Rectangle& rectangle) {
        // Draw the rectangle ...
    }

    // Additional display methods...
};
```

This separation of concerns enhances maintainability. For example, if there's a decision to support different types of screen, we only need to modify the `Canvas` class.


## Conclusion
In conclusion, the Single Responsibility Principle is a guiding principle in designing clean and maintainable software. By assigning a single responsibility to each class, we reduce complexity and enhance code flexibility. Our example of a `Rectangle` class demonstrates the pitfalls of violating SRP and the benefits of adhering to this crucial design principle. Embracing SRP fosters modular, adaptable, and robust software development practices.