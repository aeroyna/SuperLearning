# Abstraction: Interfaces and Abstract Classes - Defining Contracts and Hiding Complexity

Abstraction is a core OOP principle focusing on showing only essential information and hiding complex implementation details. It allows you to define *what* an object does rather than *how* it does it. In Java, abstraction is primarily achieved through **interfaces** and **abstract classes**. These constructs are fundamental for designing flexible, maintainable, and extensible software architectures.

## 1. Abstract Classes: Partial Implementation, Enforced Structure

An **abstract class** is a class that is declared with the `abstract` keyword. It cannot be instantiated directly (you cannot create objects of an abstract class). Abstract classes serve as a base for subclasses, providing common functionality while deferring specific implementations to their children.

### Characteristics of an Abstract Class
*   **`abstract` Keyword:** Must be declared `abstract`.
*   **No Direct Instantiation:** Cannot be created using the `new` operator (e.g., `new AbstractClass()` is illegal).
*   **Abstract Methods:** Can have `abstract` methods. An abstract method is a method that has a declaration but no implementation (no method body). Any class containing one or more abstract methods *must* be declared abstract.
*   **Concrete Methods:** Can also have regular (non-abstract) methods with full implementations. This allows sharing common code among subclasses.
*   **Constructors:** Can have constructors. These constructors are called implicitly or explicitly by the constructors of their subclasses using `super()`.
*   **Fields:** Can have any type of fields (`static`, `final`, non-`static`, non-`final`).
*   **Single Inheritance:** A class can `extend` only one abstract class, adhering to Java's single inheritance model for classes.
*   **Subclass Responsibility:** A concrete (non-abstract) subclass must override and implement *all* inherited abstract methods. If it doesn't, it too must be declared `abstract`.

### Example
```java
// Abstract class defining a common contract for shapes
abstract class Shape {
    String color; // Common attribute

    public Shape(String color) { // Constructor (used by subclasses)
        this.color = color;
    }

    // Abstract method: every concrete Shape must define how to calculate its area
    public abstract double calculateArea(); // No method body

    // Concrete method: shared behavior
    public String getColor() {
        return color;
    }

    public void displayColor() {
        System.out.println("Shape color: " + color);
    }
}

// Concrete subclass: Circle must implement calculateArea()
class Circle extends Shape {
    double radius;

    public Circle(String color, double radius) {
        super(color); // Call superclass constructor
        this.radius = radius;
    }

    @Override
    public double calculateArea() { // Implementation for Circle
        return Math.PI * radius * radius;
    }
}

// Another concrete subclass: Rectangle must implement calculateArea()
class Rectangle extends Shape {
    double length;
    double width;

    public Rectangle(String color, double length, double width) {
        super(color);
        this.length = length;
        this.width = width;
    }

    @Override
    public double calculateArea() { // Implementation for Rectangle
        return length * width;
    }
}

public class AbstractClassExample {
    public static void main(String[] args) {
        // Shape s = new Shape("Red"); // Compile-time error: Cannot instantiate abstract class

        Circle c = new Circle("Blue", 5.0);
        Rectangle r = new Rectangle("Green", 4.0, 6.0);

        c.displayColor(); // Inherited concrete method
        System.out.println("Circle area: " + String.format("%.2f", c.calculateArea())); // Implemented abstract method

        r.displayColor();
        System.out.println("Rectangle area: " + r.calculateArea());
        
        // Polymorphism with abstract classes
        Shape[] shapes = new Shape[2];
        shapes[0] = c;
        shapes[1] = r;
        for (Shape s : shapes) {
            System.out.println("Area of " + s.getColor() + " shape: " + String.format("%.2f", s.calculateArea()));
        }
    }
}
```

## 2. Interfaces: Pure Contracts, Multiple Inheritance of Type

An **interface** is a blueprint of a class. It defines a contract specifying a set of methods that a class must implement. Interfaces are used to achieve abstraction and multiple inheritance of type (a class can implement multiple interfaces).

### Characteristics of an Interface
*   **`interface` Keyword:** Declared using the `interface` keyword.
*   **No Direct Instantiation:** Cannot be instantiated directly.
*   **Methods (pre-Java 8):** All methods were implicitly `public abstract`. They had no implementation.
*   **Methods (Java 8+):** Can include `default` methods (methods with implementation), `static` methods (utility methods for the interface), and `private` methods (Java 9+).
*   **Fields:** All fields are implicitly `public static final` constants. They must be initialized.
*   **No Constructors:** Interfaces cannot have constructors.
*   **Multiple Implementation:** A class can `implement` multiple interfaces using the `implements` keyword. This allows it to inherit multiple "types" or "capabilities."
*   **Interface Extension:** An interface can `extend` multiple other interfaces.

### Example
```java
// Interface defining a common behavior
interface Flyable {
    void fly(); // Implicitly public abstract

    // Implicitly public static final constant
    int MAX_ALTITUDE = 10000; 

    // Default method (Java 8+): provides a default implementation
    default void glide() {
        System.out.println("Gliding through the air.");
    }

    // Static method (Java 8+): utility method for the interface
    static void describeFlight() {
        System.out.println("Objects that can fly move in the air.");
    }
}

// Another interface
interface Landable {
    void land();
}

// A class implementing multiple interfaces
class Bird implements Flyable, Landable {
    @Override
    public void fly() {
        System.out.println("Bird flaps its wings to fly.");
    }

    @Override
    public void land() {
        System.out.println("Bird lands gently.");
    }
    
    // Can override default methods
    @Override
    public void glide() {
        System.out.println("Bird glides gracefully.");
    }
}

public class InterfaceExample {
    public static void main(String[] args) {
        Bird eagle = new Bird();
        eagle.fly();        // Implemented method
        eagle.land();       // Implemented method
        eagle.glide();      // Default method (or overridden)
        
        Flyable.describeFlight(); // Static method called directly on interface
        System.out.println("Max altitude: " + Flyable.MAX_ALTITUDE); // Accessing constant
        
        // Polymorphism with interfaces
        Flyable anotherFlyer = new Bird(); // Reference of interface type
        anotherFlyer.fly();
    }
}
```

## 3. When to Use Abstract Class vs. Interface: A Strategic Choice

The decision between an abstract class and an interface is a fundamental design choice that impacts the extensibility and flexibility of your code.

| Feature             | Abstract Class                    | Interface                               |
| :------------------ | :-------------------------------- | :-------------------------------------- |
| **Declaration**     | `abstract class`                  | `interface`                             |
| **Instantiation**   | Cannot be instantiated directly   | Cannot be instantiated directly         |
| **Inheritance**     | `extends` (single class inheritance) | `implements` (multiple interface implementations) |
| **Methods**         | Can have **abstract** methods (no body) and **concrete** methods (with body) | Can have **abstract** methods (pre-Java 8 implicitly; Java 8+ explicitly `abstract`), `default` methods, `static` methods, `private` methods (Java 9+) |
| **Fields**          | Any type of fields (`static`, `final`, non-`static`, non-`final`) | Only `public static final` constants (implicitly) |
| **Constructors**    | Yes (used by subclasses via `super()`) | No                                      |
| **Access Modifiers**| Can use `public`, `protected`, `private`, `default` for members | All members are implicitly `public` (for abstract, default, static methods, and fields) |
| **Purpose**         | "Is-A" strong relationship (type of hierarchy). Provides common base implementation. | "Has-A" capability or contract. Defines what a class *can do*. Achieves loose coupling. |
| **Evolution**       | Easier to add new methods later (as concrete method) without breaking existing subclasses. | Adding new abstract methods before Java 8 broke all implementations. `default` methods solved this. |

### Choose an **Abstract Class** when:
*   You have a strong "is-a" relationship (e.g., `Car` is a `Vehicle`).
*   You want to provide a common base implementation for subclasses, but also require them to define their own specific behaviors (abstract methods).
*   You need to provide a default implementation for many methods, or define fields that can vary among subclasses.
*   You anticipate adding new methods to the base class in the future without breaking existing subclasses.

### Choose an **Interface** when:
*   You want to define a contract for what a class *can do*, regardless of its position in the class hierarchy.
*   You need to enable a class to have multiple "types" or "roles" (e.g., a `Car` can be `Driveable` and `Maintainable`).
*   You want to achieve loose coupling between components, allowing different implementations to be swapped out easily.
*   You want to model a capability or behavior that is independent of specific implementation details.

Both abstract classes and interfaces are powerful tools for achieving abstraction and promoting modular, flexible designs in Java, serving distinct but complementary roles in OOP.

---

### Links to Topics:
*   [Classes & Objects](01_classes_and_objects.md)
*   [Methods & Encapsulation](02_methods_and_encapsulation.md)
*   [Inheritance](03_inheritance.md)
*   [Polymorphism](04_polymorphism.md)
*   [Abstraction (Interfaces & Abstract Classes)](05_abstraction_interfaces_and_abstract_classes.md)
*   [Static & Final Keywords](06_static_and_final_keywords.md)
