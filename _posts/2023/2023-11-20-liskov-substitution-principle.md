---
layout: post
title:"Design pattern: Liskov Substitution Principle"
date: 2023-11-20 10:56:33 -0400
categories: [C++ programming language, Design Pattern]
tags: ["`C++`", ", Design Pattern", "SOLID"]
toc: true
---


In our continued exploration of SOLID principles, we've navigated through the realms of the Single Responsibility Principle (SRP) and the Open-Closed Principle (OCP). In this blog post, our focus turns to the third pillar of SOLID: the Liskov Substitution Principle (LSP), named after Barbara Liskov. We will unravel its significance in the context of crafting robust and maintainable software systems.


## The essence of Liskov Substitution Principle


The Liskov Substitution Principle (LSP) focuses on ensuring that objects of a base class can be replaced with objects of a derived class without affecting the correctness of the program.


## Realizing LSP in C++


To illustrate the challenge addressed by LSP, let's consider the rectangle and square conundrum in our simple geometric library. Recall that we have developed a `Rectangle` class managing a rectangle's properties (i.e., height and withd) and calculation (e.g., area calculation.) Now, the need arises to create a `Square` class derived from the `Rectangle` class:


```C++
class Rectangle : public Geometry {
public:
    Rectangle(int height, int width) : mHeight(height), mWidth(width) {}

    virtual void setHeight(int height) {
        mHeight = height;
    }

    virtual void setWidth(int width) {
        mWidth = width;
    }

    int getHeight() const {
        return mHeight;
    }

    int getWidth() const {
        return mWidth;
    }

    int calculateArea() const override {
        return mWidth * mHeight;
    }

private:
    int mHeight;
    int mWidth;
};

class Square: public Rectangle {
public:
    Square(int size) : Rectangle(size, size) {}

    // Set both height and witdh to the new value
    void setHeight(int height) override {
        Rectangle::setHeight(height);
        Rectangle::setWidth(height);
    }

    // Set both height and witdh to the new value
    void setWidth(int width) override {
        Rectangle::setHeight(width);
        Rectangle::setWidth(width);
    }
};
```


While this implementation may seem correct mathematically (as a square is a type of rectangle), it violates the LSP. Let's consider a situation where we have a function designed for rectangle and adjusting only the height:


```C++
void process(Rectangle &rectangle) {
    int width = rectangle.getWidth()
    rectangle.setHeight(10);

    assert(rectangle.calculateArea() == width * 10)

    // Additional processing...
}
}

int main() {
    Rectangle rectangle(2, 3);
    Square square(2);

    process(rectangle); // Correct
    process(square);    // Violates the assert
}
```


To adhere to the LSP, the `Square` class should not derived from the `Rectangle` class but from the `Geometry` class to manage it owns properties and calculations.

```C++
class Square: public Geometry {
private:
    int mSize;
public:
    // Methods for managing properties and calculations.
};
```


## Conclusion


In the world of object-oriented design, the Liskov Substitution Principle serves as a guiding light, ensuring that derived classes can seamlessly replace their base classes. Violating this principle can lead to unexpected behavior and compromise the integrity of a software system. By adhering to the principle, developers foster code reuse, simplify the maintenance process, and advoid potential bugs.