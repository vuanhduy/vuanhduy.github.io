---
layout: post
title:"Design pattern: Open-Closed Principle"
date: 2023-11-17 10:56:33 -0400
categories: [C++ programming language, Design Pattern]
tags: ["`C++`", ", Design Pattern", "SOLID"]
toc: true
---


In our exploration of SOLID design principles, we previously delved into the Single Responsibility Principle (SRP), emphasizing the allocation of a single responsibility to each class. Our current focus shifts to the Open-Closed Principle (OCP), which directs our attention towards the broader aspects of extensibility and adaptability within our codebase. This blog post examines the significance of OCP through a practical example in C++.


## What is the Open-Closed Principle?
OCP emphasizes that a system should be open for extension but closed for modification. In essence, the existing codebase should remain untouched when introducing new features or functionalities.


## Realizing OCP in C++
To illustrate the challenge addressed by OCP, let's revisit our SRP's example involving a simplistic geometric library. Previously, we have developed a `Rectangle` class handling a rectangle's properties and calculations, and `Canvas` class responsible for drawing rectangles. As requirements eveolve, new shapes, such as `Circle` and `Triagle`, are added. This leads to continuous modifications of the `Canvas` class, resulting in maintenance challenges and potential bugs.


```C++
class Circle {
    // Managing a circle's properties and calculations
};

class Triangle {
    // Managing a triangle's properties and calculations
};

class Canvas {
public:
    static void draw(const Rectangle& rectangle) {
        // Draw the rectangle ...
    }

    static void draw(const Circle& circle) {
        // Draw the circle ...
    }

    static void draw(const Triangle& triangle) {
        // Draw the triangle ...
    }

    // Additional display methods...
};
```


The Open-Closed Principle provides a more elegant solution through abstract and derived classes:
   - `Geometry` is an abstract class defining common properties and calculations for geometric shape, such as calculating area.
   - `Rectangle`, `Circle`, and `Triangle` are derived from the `Geometry` class, managing shapes' specific properties, and calculations.
   - `Canvas` is an abstract class defining common methods for drawing geometric shapes.
   - `RectangleDrawer`, `CircleDrawer`, and `TriangleDrawer` are derived from the `Canvas` class to provide specific drawing logic for rectangles, circles, and triangle, respectively.


```C++
class Geometry {
public:
    virtual calculateArea() const = 0;
    // Additional pure virtual methods for managing calculations ...
}

class Rectangle: public Geometry {
    // Managing a rectangle's properties and calculations
}

class Circle: public Geometry  {
    // Managing a circle's properties and calculations
};

class Triangle: public Geometry  {
    // Managing a triangle's properties and calculations
};

class Canvas {
public:
    virtual void draw(const Geometry& geometry) const = 0;
    // Additional drawing methods
}

class RectangleDrawer: public Canvas {
public:
    void draw(const Geometry& geometry) const {
        // Drawing logic for a rectangle ...
    }

    // Additional drawing methods
}

class CircleDrawer: public Canvas {
public:
    void draw(const Geometry& geometry) const {
        // Drawing logic for a circle ...
    }

    // Additional drawing methods
}

class TriangleDrawer: public Canvas {
public:
    void draw(const Geometry& geometry) const {
        // Drawing logic for a triangle ...
    }

    // Additional drawing methods
}

int main() {
    Rectangle rectangle(...);
    Circle circle(...);
    Triangle triangle(...);

    std::unique_ptr<Drawer> ptrRectangleDrawer(new RectangleDrawer);
    std::unique_ptr<Drawer> ptrCircleDrawer(new CircleDrawer);
    std::unique_ptr<Drawer> ptrTriangleDrawer(new TriangleDrawer);

    std::cout << "Drawing Rectangle:" << std::endl;
    ptrRectangleDrawer->draw_geometry(rectangle);

    std::cout << "\nDrawing Circle:" << std::endl;
    ptrCircleDrawer->draw_geometry(circle);

    std::cout << "Drawing Triangle:" << std::endl;
    ptrTriangleDrawer->draw_geometry(triangle);

    return 0;
}
```

With these components in place, drawing geomytric shape becomes more flexible and extensible. We can easily add new shape and drawer without modifying existing code.


## Conclusion


The Open-Closed Principle advocates for systems that are open to extension but closed for modification. By incorporating new abstract classes and adhering to the principles of OCP, we developed a more resilient system capable of gracefully evolving with changing requirements. This approach enhances code maintainability, scalability, and robustness, laying the foundation for a more resilient and adaptable software architecture.