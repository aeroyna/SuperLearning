# Classes and Objects

Java is a purely **Object-Oriented Programming (OOP)** language (except for its primitive types). At its core, OOP is about modeling real-world entities as software components.

## 1. What is a Class?
A class is a **blueprint** or a **template** for creating objects. It defines the characteristics (attributes/properties/fields) and behaviors (actions/methods) that an object of that class will have.
*   **Analogy:** A class is like the blueprint for a house. It describes what a house *should* have (number of rooms, windows, doors), but it's not an actual house itself.

### Class Declaration
```java
// public access modifier (can be accessed from anywhere)
public class Dog { 
    // Attributes (State/Properties/Fields)
    String breed;
    String name;
    int age;

    // Behaviors (Methods)
    public void bark() {
        System.out.println(name + " barks!");
    }

    public void eat(String food) {
        System.out.println(name + " is eating " + food);
    }
}
```

## 2. What is an Object?
An object is an **instance** of a class. It's a concrete entity created from the class blueprint. Each object has its own unique set of attribute values.
*   **Analogy:** An object is like an actual house built from the blueprint. Each house is unique (different color, different occupants) but follows the same blueprint.

### Creating an Object (Instantiation)
Objects are created using the `new` keyword, which invokes a **constructor**.
```java
// Declaring a reference variable of type Dog
Dog myDog; 

// Creating an object (instantiation) and assigning its reference to myDog
myDog = new Dog(); 
// Alternatively, declaration and instantiation in one line:
Dog yourDog = new Dog();
```

### Accessing Attributes and Methods
Use the **dot operator (`.`)** to access an object's fields and methods.
```java
Dog myDog = new Dog();

// Assigning values to attributes (state)
myDog.breed = "Golden Retriever";
myDog.name = "Buddy";
myDog.age = 3;

// Calling methods (behavior)
myDog.bark();        // Output: Buddy barks!
myDog.eat("kibble"); // Output: Buddy is eating kibble

System.out.println(myDog.name + " is a " + myDog.breed);
// Output: Buddy is a Golden Retriever
```

## 3. Constructors
A constructor is a special type of method that is automatically called when an object is created using the `new` keyword. Its primary purpose is to initialize the object's state (its instance variables).

### Characteristics of a Constructor
*   **Same Name as Class:** Must have the exact same name as the class.
*   **No Return Type:** Does not have a return type (not even `void`).
*   **Access Modifiers:** Can have `public`, `private`, `protected`, or default access.

### Default Constructor
If you don't define any constructor, Java provides a **default (no-argument) constructor** implicitly. It initializes instance variables to their default values (e.g., 0 for numbers, `false` for boolean, `null` for objects).

### No-Argument Constructor (Explicitly Defined)
```java
public class Car {
    String make;
    String model;

    // Explicit no-argument constructor
    public Car() {
        this.make = "Unknown";
        this.model = "Unknown";
        System.out.println("Car object created!");
    }
}

Car myCar = new Car(); // Calls the no-argument constructor
// Output: Car object created!
```

### Parameterized Constructor
Constructors can take parameters to initialize an object with specific values at the time of creation.
```java
public class Car {
    String make;
    String model;

    // Parameterized constructor
    public Car(String make, String model) {
        this.make = make;   // 'this' refers to the current object's instance variable
        this.model = model;
    }
}

Car yourCar = new Car("Toyota", "Camry"); // Calls parameterized constructor
System.out.println(yourCar.make + " " + yourCar.model); // Output: Toyota Camry
```

### `this` Keyword
Inside a constructor or method, `this` refers to the current object. It's often used to distinguish between an instance variable and a parameter with the same name.
```java
public class Person {
    String name;
    int age;

    public Person(String name, int age) {
        this.name = name; // 'this.name' refers to the instance variable
        this.age = age;   // 'name' and 'age' without 'this' refer to parameters
    }
}
```
*   `this()`: Can also be used inside one constructor to call another constructor of the same class (constructor chaining). It must be the first statement in the constructor.

## 4. `null` Reference
When an object reference variable is declared but not yet assigned an object, or if it's explicitly set to `null`, it doesn't point to any object in memory.
```java
Dog anotherDog = null;
// anotherDog.bark(); // This would cause a NullPointerException at runtime
```
*   Accessing a method or field of a `null` reference will result in a **`NullPointerException`**, one of Java's most common runtime errors.

---
## 5. Classes as Custom Data Types
You can think of classes as custom data types. Just as `int` defines an integer, `Dog` defines a Dog object. You can create variables of type `Dog`, pass `Dog` objects to methods, and return `Dog` objects from methods.
This is a cornerstone of code organization and reusability in Java.