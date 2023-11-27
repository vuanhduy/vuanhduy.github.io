---
layout: post
title:"Design pattern: Interface Segregation Principle"
date: 2023-11-22 10:56:33 -0400
categories: [C++ programming language, Design Pattern]
tags: ["`C++`", ", Design Pattern", "SOLID"]
toc: true
---


In our ongoing exploration of SOLID design principles, we have previously explored the Single Responsibility Principle (SRP), the Open-Closed Principle (OCP), and the Liskov Substitution Principle (LSP). Now, let's delve into the fourth principle: the Interface Segregation Principle (ISP).


## Understanding Interface Segregation Principle


The Interface Segregation Principle focuses on interfaces design and advocates for the creation of specific, client-focused interfaces rather than large, all-encompassing ones. It emphasizes the importance of keeping interfaces lean and mean, tailored to the precise needs of the client, thereby avoiding the implementation of unnecessary methods. Violating ISP often leads to the creation of "fat" interfaces, where implementing classes are forced to provide implementations for methods that are irrelevant to their functionalities. Adhering to ISP, on the other hand, involves breaking down large interfaces into smaller, more specialized ones, ensuring that classes only implement methods relevant to their specific use cases. 


At first glance, this principle seems akin the SRP, but they are distinct. The ISP encourages to break a class, which have already satisfied the SRP, into smaller more specific ones. This ensures that derived classes do not have to implement unnecessary methods. To illustrate this, let's revisit our simple graphical library that manages shapes' properties and calculations (i.e., `Shape`, `Rectangle`, `Circles`, ...) and renders them (i.e., `Canvas`, `RectangleDrawer`, `CirclesDrawer`, ...). As requirements evolves to include rendering strings, a naive approach would be is to extend the `Canvas` interface:


```C++
class Canvas {
public:
    virtual void draw(const Geometry& geometry) const = 0;
    virtual void draw(const string& str) const = 0;

    // Additional drawing methods
}
```


Although this adheres the SRP, it violates the ISP as detail classes (e.g., `RectangleDrawer`, `CircleDrawer`) have to implement unsused methods (e.g., `void draw(const string &str)`), which normally throw exceptions. To adhere to the ISP, we should devide the interface to smaller ones (`GeometryDrawer` and `StringDrawer`). The revised implementation is as follows:


```C++
class Canvas {
public:
    virtual void resize(...) const = 0;
    // Additional common draw methods
};

class GeometryDrawer {
public:
    virtual void draw(const Geometry& geometry) const = 0;
    // Additional draw methods for geometric shapes
};

class StringDrawer {
public:
    virtual void draw(const string& str) const = 0;
    // Additional draw methods for strings
};

class RectangleDrawer: public Canvas, public GeometryDrawer {
public:
    void draw(const Geometry& geometry) const {
        // Drawing logic for a rectangle ...
    }

    // Additional drawing methods
};

// Different drawer classes for different string styles
class TimesNewRomanStringDrawer: public Canvas, public StringDrawer {
public:
    void draw(const string &std) const {
        // Drawing logic for the string
    }

    // Additional drawing methods for strings
}
```


## Conclusion


The Interface Segregation Principle guides us to keep our interfaces lean and focused. By breaking down monolithic interfaces into smaller, cohesive ones, we create a more flexible and understandable design. This principle not only improves code quality but also enhances the overall maintainability and extensibility of our software. As we continue our exploration of design principles, remember that a well-segregated interface is a key building block for robust and adaptable systems.