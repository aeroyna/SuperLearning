# Abstraction

Abstraction is the concept of hiding the complex implementation details and showing only the essential features of the object. It helps in managing complexity by allowing us to focus on what an object does instead of how it does it.

## How to achieve Abstraction?

Abstraction can be achieved in two ways:

1.  **Abstraction using Classes:** We can implement abstraction using classes. The class helps us to group data members and member functions using available access specifiers. A class can decide which data member will be visible to the outside world and which is not.

2.  **Abstraction using Header Files:** For example, we use the `pow()` function from the `<cmath>` header file without knowing its implementation details.

## Abstraction vs. Encapsulation

Abstraction and encapsulation are related but distinct concepts.

| Abstraction                                     | Encapsulation                               |
|-------------------------------------------------|---------------------------------------------|
| Hides complexity by showing only what is necessary. | Hides data by bundling it with methods.    |
| Focuses on the "what".                          | Focuses on the "how".                       |
| Implemented using abstract classes and interfaces. | Implemented using access specifiers (`private`, `protected`). |
| Solves the problem at the design level.         | Solves the problem at the implementation level. |

### Example

Let's consider a real-world example of a car.

*   **Abstraction:** You don't need to know how the engine works to drive a car. You just need to know how to use the steering wheel, pedals, and gear stick. The complex internal mechanism is hidden from you.

*   **Encapsulation:** The engine, transmission, and other components are all self-contained units that are assembled to create the car. You cannot directly access the individual parts of the engine.

In C++, abstraction is often achieved using **abstract classes**. As we saw in the section on polymorphism, an abstract class defines an interface (a set of methods) but does not provide the implementation. The implementation is left to the derived classes.

```cpp
// Abstract class representing the concept of a "Shape"
class Shape {
public:
    virtual double getArea() = 0; // what it does, not how it does it
};

// Concrete class providing the implementation
class Rectangle : public Shape {
private:
    double width, height;
public:
    Rectangle(double w, double h) : width(w), height(h) {}
    double getArea() override {
        return width * height; // how it does it
    }
};
```

Here, the `Shape` class represents the abstract concept of a shape that has an area. The user of the `Shape` class does not need to know how the area is calculated for a specific shape.
