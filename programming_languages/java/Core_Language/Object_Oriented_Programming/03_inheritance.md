# Inheritance: Building Class Hierarchies for Reusability

Inheritance is a foundational principle of Object-Oriented Programming (OOP) that allows a class to inherit properties (fields) and behaviors (methods) from another class. This mechanism is crucial for promoting **code reusability**, reducing redundancy, and establishing a natural "is-a" relationship between classes.

## 1. The "Is-A" Relationship: Understanding the Link

Inheritance creates a hierarchical relationship between classes:
*   **Superclass (Parent/Base Class):** The class whose features are inherited. It defines common attributes and behaviors.
*   **Subclass (Child/Derived Class):** The class that inherits the features from the superclass and can add its own specific attributes and behaviors.

The "is-a" test is a good heuristic: If `X` "is-a" `Y`, then `X` can extend `Y`.
*   A `Car` **is-a** `Vehicle`. (So `Car extends Vehicle`).
*   A `Dog` **is-a** `Animal`. (So `Dog extends Animal`).

## 2. The `extends` Keyword: Establishing Inheritance

In Java, class inheritance is achieved using the `extends` keyword.

### Syntax
```java
class Subclass extends Superclass {
    // fields and methods specific to Subclass
    // Plus, all non-private members from Superclass are available
}
```

### Example
```java
// Superclass: Defines common attributes and behaviors for all animals
class Animal {
    String name;
    int age;

    public Animal(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public void eat() {
        System.out.println(name + " is eating.");
    }

    public void sleep() {
        System.out.println(name + " is sleeping.");
    }

    public String getInfo() {
        return name + ", " + age + " years old.";
    }
}

// Subclass: Dog inherits from Animal and adds its own specific characteristics
class Dog extends Animal {
    String breed;

    public Dog(String name, int age, String breed) {
        super(name, age); // Calls the constructor of the superclass (Animal)
        this.breed = breed;
    }

    public void bark() {
        System.out.println(name + " barks loudly!");
    }

    // A method specific to Dog, using inherited 'name'
    public String getBreedInfo() {
        return name + " is a " + breed + ".";
    }
}

// Usage
public class InheritanceExample {
    public static void main(String[] args) {
        Dog myDog = new Dog("Buddy", 3, "Golden Retriever");

        // Accessing inherited fields and methods
        System.out.println(myDog.name);  // Inherited
        myDog.eat();                     // Inherited method
        myDog.sleep();                   // Inherited method

        // Accessing subclass-specific fields and methods
        System.out.println(myDog.breed); // Specific to Dog
        myDog.bark();                    // Specific to Dog
        
        System.out.println(myDog.getInfo()); // Inherited method, potentially overridden
        System.out.println(myDog.getBreedInfo());
    }
}
```
*   **Access to Members:** A subclass inherits all `public` and `protected` members of its superclass. `private` members are not directly accessible but can be accessed indirectly via `public` or `protected` getter/setter methods in the superclass.

## 3. Method Overriding: Customizing Inherited Behavior

Method overriding occurs when a subclass provides its own specific implementation for a method that is already defined (and inherited) in its superclass. This is a key aspect of runtime polymorphism.

### Rules for Overriding
*   **Same Method Signature:** The overriding method in the subclass must have the exact same name, number and type of parameters, and order of parameters as the method in the superclass.
*   **Covariant Return Type:** The return type must be the same or a subtype (covariant return type) of the return type declared in the superclass method (since Java 5).
*   **Access Modifier:** The access modifier of the overriding method cannot be more restrictive than the overridden method's access modifier (e.g., a `protected` method cannot be overridden by a `private` method).
*   **Exception Handling:** The overriding method cannot throw new or broader checked exceptions than the overridden method.
*   **`@Override` Annotation:** It's highly recommended to use the `@Override` annotation above an overriding method. This is a marker annotation that tells the compiler you intend to override a method. If you accidentally violate any overriding rules (e.g., a typo in the method name or wrong parameters), the compiler will generate an error, preventing subtle bugs.

### Example
```java
class Animal {
    public void makeSound() {
        System.out.println("Animal makes a generic sound.");
    }
}

class Dog extends Animal {
    @Override // Good practice: compiler will verify if this truly overrides a method
    public void makeSound() { // Same signature as in Animal
        System.out.println("Dog barks: Woof!");
    }
}

class Cat extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Cat meows: Meow!");
    }
}

// Usage
public class MethodOverridingExample {
    public static void main(String[] args) {
        Animal myAnimal = new Animal();
        myAnimal.makeSound(); // Output: Animal makes a generic sound.

        Dog myDog = new Dog();
        myDog.makeSound();    // Output: Dog barks: Woof!

        Cat myCat = new Cat();
        myCat.makeSound();    // Output: Cat meows: Meow!
    }
}
```

## 4. The `super` Keyword: Referring to the Parent

The `super` keyword is a special reference variable used to refer to the immediate superclass object.

### Uses of `super`
1.  **To invoke superclass constructor:**
    *   `super()` (calls the no-arg constructor of the superclass).
    *   `super(parameters)` (calls a specific parameterized constructor of the superclass).
    *   **Rule:** `super()` or `super(parameters)` must be the **first statement** in the subclass constructor. If you don't explicitly call `super()` or `super(...)`, Java automatically inserts an implicit `super()` call.

2.  **To invoke superclass method:** `super.methodName()`. This is useful when you want to extend the superclass's method behavior rather than completely replace it.

3.  **To access superclass field:** `super.fieldName`. This is used if the subclass has a field with the same name as an inherited field, and you need to refer to the superclass's version.

### Example
```java
class Vehicle {
    String brand;
    public Vehicle(String brand) { this.brand = brand; }
    public void start() {
        System.out.println(brand + " vehicle starts.");
    }
}

class Car extends Vehicle {
    int year;

    public Car(String brand, int year) {
        super(brand); // Calls Vehicle's constructor (must be first!)
        this.year = year;
    }

    @Override
    public void start() {
        super.start(); // Calls the start() method from Vehicle
        System.out.println("Car specific start sequence initiated for " + brand + " (" + year + ").");
    }

    public void displayBrandInfo() {
        System.out.println("Car's brand: " + brand); // Refers to Car's inherited 'brand'
        System.out.println("Vehicle's brand (via super): " + super.brand); // Explicitly refers to Superclass's 'brand'
    }
}

public class SuperKeywordExample {
    public static void main(String[] args) {
        Car myCar = new Car("Honda", 2023);
        myCar.start();
        myCar.displayBrandInfo();
    }
}
```

## 5. Single Inheritance in Java Classes

Java supports **single inheritance** for classes. This means a class can `extend` only one superclass.
*   **Reason:** To avoid the "Diamond Problem" (ambiguity that arises in languages supporting multiple inheritance, where a class inherits methods from two parent classes that both inherited from a common grandparent, leading to uncertainty about which method implementation to use).

### Multiple Inheritance of Interface Type
While classes can only extend one class, a class can `implement` multiple **interfaces**. This is often referred to as "multiple inheritance of type" because the class inherits multiple contracts (method signatures) but not their implementations (prior to default methods in Java 8).

## 6. The `final` Keyword and Inheritance

The `final` keyword can be used to control aspects of inheritance:
*   **`final class`:** A class declared `final` cannot be subclassed (e.g., `java.lang.String`, `java.lang.Math` are final classes). This is used for security (ensuring behavior cannot be altered) or to ensure immutability.
*   **`final method`:** A method declared `final` cannot be overridden by subclasses. This ensures that the method's implementation remains consistent across the entire hierarchy.
*   **`final field`:** A field declared `final` (instance or static) must be initialized once and its value cannot be changed afterwards. This applies to primitive values directly and to object references (the reference cannot change, but the object it points to *can* change if it's mutable).

Inheritance is a powerful tool for designing extensible and reusable software, forming the backbone of class hierarchies in complex applications.

---

### Links to Topics:
*   [Classes & Objects](01_classes_and_objects.md)
*   [Methods & Encapsulation](02_methods_and_encapsulation.md)
*   [Inheritance](03_inheritance.md)
*   [Polymorphism](04_polymorphism.md)
*   [Abstraction (Interfaces & Abstract Classes)](05_abstraction_interfaces_and_abstract_classes.md)
*   [Static & Final Keywords](06_static_and_final_keywords.md)
