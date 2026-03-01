# Classes and Objects

Object-Oriented Programming (OOP) is a programming paradigm based on the concept of "objects", which can contain data in the form of fields (often known as attributes or properties) and code in the form of procedures (often known as methods).

## Classes

A class is a blueprint for creating objects. It defines a set of attributes and methods that the created objects will have.

### Syntax

```cpp
class ClassName {
public:
    // public members (attributes and methods)

private:
    // private members

protected:
    // protected members
};
```

*   **`public`:** Members are accessible from outside the class.
*   **`private`:** Members are not accessible from outside the class. This is the default access specifier.
*   **`protected`:** Members are not accessible from outside the class, but they are accessible in inherited classes.

## Objects

An object is an instance of a class. When a class is defined, no memory is allocated. Memory is allocated when an object is created.

### Creating Objects

```cpp
ClassName objectName;
```

### Example: A `Dog` class

```cpp
#include <iostream>
#include <string>

class Dog {
public:
    // Attribute
    std::string breed;

    // Method
    void bark() {
        std::cout << "Woof!" << std::endl;
    }
};

int main() {
    Dog myDog; // create an object of the Dog class

    myDog.breed = "Golden Retriever";
    std::cout << "My dog's breed is: " << myDog.breed << std::endl;

    myDog.bark();

    return 0;
}
```

## Member Functions

Member functions can be defined inside or outside the class definition.

### Inside the class definition

```cpp
class MyClass {
public:
    void myMethod() {
        std::cout << "Hello from inside the class!" << std::endl;
    }
};
```

### Outside the class definition

To define a member function outside the class, you use the scope resolution operator (`::`).

```cpp
class MyClass {
public:
    void myMethod(); // declaration
};

void MyClass::myMethod() { // definition
    std::cout << "Hello from outside the class!" << std::endl;
}
```
This is often done to separate the interface (in a header file) from the implementation (in a source file).
