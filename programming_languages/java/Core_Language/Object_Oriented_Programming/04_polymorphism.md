# Polymorphism: Embracing Multiple Forms

Polymorphism, one of the four pillars of Object-Oriented Programming (OOP), means "many forms" (from Greek "poly" - many, "morph" - form). In Java, it refers to the ability of objects of different classes to be treated as objects of a common superclass or interface type. This allows for writing flexible, reusable, and extensible code that can adapt to different data types at runtime.

Java primarily achieves polymorphism through **method overloading** (compile-time polymorphism) and **method overriding** (runtime polymorphism).

## 1. Compile-time Polymorphism (Method Overloading)

*   **Definition:** Occurs when a class has multiple methods with the **same name** but **different parameter lists**.
*   **Resolution:** The Java compiler determines which overloaded method to call based on the number, type, and order of arguments provided at **compile time**.
*   **Characteristics:**
    *   Methods must share the same name.
    *   They must differ in the number of parameters, the data types of the parameters, or the order of the data types of the parameters.
    *   The return type alone is **not sufficient** to distinguish overloaded methods.
    *   Also known as **Static Polymorphism** or **Early Binding**.

### Example (from previous Methods chapter, now in context of Polymorphism)
```java
class Calculator {
    // Overloaded add methods
    public int add(int a, int b) { return a + b; }
    public double add(double a, double b) { return a + b; }
    public String add(String a, String b) { return a + b; } // String concatenation

    public static void main(String[] args) {
        Calculator calc = new Calculator();
        System.out.println(calc.add(10, 20));             // Calls add(int, int) -> 30
        System.out.println(calc.add(10.5, 20.5));         // Calls add(double, double) -> 31.0
        System.out.println(calc.add("Hello", "World")); // Calls add(String, String) -> "HelloWorld"
    }
}
```
The compiler looks at the arguments (`int`, `double`, `String`) and matches them to the appropriate `add` method signature during compilation.

## 2. Runtime Polymorphism (Method Overriding)

*   **Definition:** Occurs when a subclass provides its own specific implementation for a method that is already defined in its superclass.
*   **Resolution:** The Java Virtual Machine (JVM) determines which method implementation to call based on the actual object type (not the reference type) at **runtime**.
*   **Characteristics:**
    *   Requires an "is-a" relationship (inheritance).
    *   The method in the subclass must have the exact same name, return type (or a covariant return type), and parameter list as the method in the superclass.
    *   The access modifier of the overriding method cannot be more restrictive than that of the overridden method.
    *   **Crucial:** The object *reference* can be of the supertype, but the *actual object* in memory is of the subtype.
    *   Also known as **Dynamic Polymorphism** or **Late Binding**.

### Example
```java
class Animal {
    public void makeSound() {
        System.out.println("Animal makes a generic sound.");
    }
}

class Dog extends Animal {
    @Override
    public void makeSound() { // Overriding Animal's makeSound()
        System.out.println("Dog barks!");
    }
    public void fetch() { System.out.println("Dog fetches."); }
}

class Cat extends Animal {
    @Override
    public void makeSound() { // Overriding Animal's makeSound()
        System.out.println("Cat meows.");
    }
    public void scratch() { System.out.println("Cat scratches."); }
}

public class RuntimePolymorphismExample {
    public static void main(String[] args) {
        // Polymorphic references: Animal references pointing to subtype objects
        Animal myAnimal1 = new Dog();    // An Animal reference pointing to a Dog object
        Animal myAnimal2 = new Cat();    // An Animal reference pointing to a Cat object

        // Calling a method through a polymorphic reference
        myAnimal1.makeSound(); // Output: Dog barks! (JVM calls Dog's makeSound() at runtime)
        myAnimal2.makeSound(); // Output: Cat meows. (JVM calls Cat's makeSound() at runtime)

        // What methods can be called?
        // myAnimal1.fetch(); // Compile-time error: Animal reference has no fetch() method
        // To call fetch(), you would need to downcast:
        if (myAnimal1 instanceof Dog) {
            Dog d = (Dog) myAnimal1;
            d.fetch();
        }
    }
}
```
In this example, `myAnimal1` and `myAnimal2` are both declared as type `Animal`, but the JVM correctly calls the `makeSound()` method of the actual object type (`Dog` or `Cat`) at runtime. This allows you to write generic code that operates on `Animal` objects, and the correct behavior is invoked depending on the specific type of animal.

### Covariant Return Types (Java 5+)
Prior to Java 5, the return type of an overriding method had to be *identical* to the overridden method. Now, it can be a *subtype* of the original return type. This is called a covariant return type.

```java
class Vehicle {
    public Vehicle getVehicle() { return new Vehicle(); }
}

class Car extends Vehicle {
    @Override
    public Car getVehicle() { return new Car(); } // 'Car' is a subtype of 'Vehicle'
}
```

## 3. Polymorphism with Interfaces and Abstract Classes

Polymorphism is not limited to class inheritance. It is equally, if not more, powerfully applied when dealing with **Interfaces** and **Abstract Classes**. An object can be referred to by the type of any interface it implements or any abstract class it extends. This allows for even greater flexibility, as a class can implement multiple interfaces.

### Example with Interface (Conceptual)
```java
interface Shape {
    void draw(); // All implementing classes must provide this method
}

class Circle implements Shape {
    @Override public void draw() { System.out.println("Drawing Circle"); }
}

class Rectangle implements Shape {
    @Override public void draw() { System.out.println("Drawing Rectangle"); }
}

public class InterfacePolymorphismExample {
    public static void main(String[] args) {
        // Polymorphic array: an array of Shape references
        Shape[] shapes = new Shape[2];
        shapes[0] = new Circle();    // Shape reference, Circle object
        shapes[1] = new Rectangle(); // Shape reference, Rectangle object

        for (Shape s : shapes) {
            s.draw(); // Calls Circle's draw() or Rectangle's draw() at runtime
        }
        // Output:
        // Drawing Circle
        // Drawing Rectangle
    }
}
```
This is a powerful pattern for designing extensible systems: a new shape type can be added by simply implementing the `Shape` interface, and the `for` loop will automatically handle it without modification.

## 4. `instanceof` Operator and Pattern Matching (Java 16+)

The `instanceof` operator is used to check if an object is an instance of a particular class or an interface.

```java
Animal a = new Dog();
if (a instanceof Dog) {
    Dog d = (Dog) a; // Explicit Downcasting needed
    d.fetch();
}
```
*   **Nuance:** `instanceof` checks the actual type of the object, not just the reference type.

### Pattern Matching for `instanceof` (Java 16+)
This feature simplifies the common `instanceof` check followed by a cast.

```java
Object obj = new Dog();
if (obj instanceof Dog d) { // 'd' is the pattern variable, automatically cast
    d.fetch(); // Can directly use 'd' here, it's already a Dog
} else if (obj instanceof Cat c) {
    c.scratch();
}
```
This makes code much cleaner and less error-prone.

## 5. Downcasting and Upcasting: Navigating the Hierarchy

### Upcasting (Implicit and Safe)
Treating a subclass object as its superclass type. This is always safe because a subclass *is-a* superclass. It happens implicitly.

```java
Dog myDog = new Dog("Buddy", 3, "Golden");
Animal myAnimal = myDog; // Upcasting - myDog (a Dog object) is treated as an Animal. Always safe.
```
*   You lose access to subclass-specific methods (e.g., `myAnimal.bark()` is a compile-time error).

### Downcasting (Explicit and Risky)
Treating a superclass reference as a subclass type. This requires an explicit cast `(SubClass)`. It is risky because if the actual object in memory is not of the target subclass type, a `ClassCastException` will be thrown at runtime.

```java
Animal myAnimal = new Dog("Max", 5, "Labrador"); // Actual object is Dog
// Dog anotherDog = (Dog) myAnimal; // Downcasting - Safe because myAnimal actually IS a Dog
// anotherDog.bark();

Animal yetAnotherAnimal = new Cat("Whiskers", 2, "Siamese"); // Actual object is Cat
// Dog problemDog = (Dog) yetAnotherAnimal; // RUNTIME ERROR: ClassCastException!
                                          // A Cat cannot be cast to a Dog
```
*   **Best Practice:** Always use `instanceof` (or pattern matching) before attempting a downcast to prevent `ClassCastException`. However, good object-oriented design often aims to **minimize the need for downcasting** by leveraging polymorphism. If you find yourself downcasting frequently, it might indicate a flaw in your class hierarchy design.

Polymorphism is a cornerstone of flexible and reusable software development in Java, enabling code that can adapt to different types and behaviors at runtime, making systems more extensible and easier to manage.

---

### Links to Topics:
*   [Classes & Objects](01_classes_and_objects.md)
*   [Methods & Encapsulation](02_methods_and_encapsulation.md)
*   [Inheritance](03_inheritance.md)
*   [Polymorphism](04_polymorphism.md)
*   [Abstraction (Interfaces & Abstract Classes)](05_abstraction_interfaces_and_abstract_classes.md)
*   [Static & Final Keywords](06_static_and_final_keywords.md)
