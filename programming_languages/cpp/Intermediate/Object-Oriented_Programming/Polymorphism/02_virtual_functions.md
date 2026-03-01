# Virtual Functions

A virtual function is a member function that you expect to be redefined in derived classes. When you refer to a derived class object using a pointer or a reference to the base class, you can call a virtual function for that object and execute the derived class's version of the function.

To make a function virtual, you use the `virtual` keyword.

### Example with `virtual`

Let's modify the previous example to use a virtual function.

```cpp
#include <iostream>

class Animal {
public:
    virtual void speak() {
        std::cout << "Animal speaks" << std::endl;
    }
};

class Dog : public Animal {
public:
    void speak() override { // 'override' is optional but good practice
        std::cout << "Dog barks" << std::endl;
    }
};

class Cat : public Animal {
public:
    void speak() override {
        std::cout << "Cat meows" << std::endl;
    }
};

int main() {
    Animal* a1 = new Dog();
    Animal* a2 = new Cat();

    a1->speak(); // Now calls Dog::speak()
    a2->speak(); // Now calls Cat::speak()

    delete a1;
    delete a2;
    return 0;
}
```

Now, the output is as expected:
```
Dog barks
Cat meows
```

```
Dog barks
Cat meows
```

## How Virtual Functions Work (V-Table)

When a class contains a virtual function, the compiler creates a **V-Table** (Virtual Table) for that class. This table contains pointers to the virtual functions for that class. Each object of the class contains a hidden pointer (vptr) that points to the V-Table.

```mermaid
graph LR
    subgraph Objects
        DogObj[Dog Object] -- vptr --> DogVTable
        CatObj[Cat Object] -- vptr --> CatVTable
    end

    subgraph VTables
        DogVTable[Dog V-Table]
        CatVTable[Cat V-Table]
        
        DogVTable -- speak() --> DogSpeak[Dog::speak()]
        CatVTable -- speak() --> CatSpeak[Cat::speak()]
    end
    
    style DogVTable fill:#e3f2fd
    style CatVTable fill:#fff3e0
```

## `override` Keyword (C++11 and later)


The `override` keyword is used to indicate that a function in a derived class is intended to override a virtual function in the base class. It helps prevent bugs, for example, if you misspell the function name or if the parameters don't match. The compiler will generate an error if the function doesn't actually override a base class function.

## Abstract Classes and Pure Virtual Functions

An **abstract class** is a class that is designed to be specifically used as a base class. Abstract classes cannot be instantiated.

An abstract class is a class that has at least one **pure virtual function**. A pure virtual function is a virtual function that has no implementation in the base class. You declare a pure virtual function by assigning `= 0`.

```cpp
class Shape {
public:
    virtual double getArea() = 0; // pure virtual function
};
```

Any class that inherits from `Shape` must provide an implementation for `getArea()`, or it will also be an abstract class.

### Example with Abstract Class

```cpp
#include <iostream>

// Abstract base class
class Shape {
public:
    virtual double getArea() = 0;
};

class Rectangle : public Shape {
private:
    double width, height;
public:
    Rectangle(double w, double h) : width(w), height(h) {}
    double getArea() override {
        return width * height;
    }
};

class Circle : public Shape {
private:
    double radius;
public:
    Circle(double r) : radius(r) {}
    double getArea() override {
        return 3.14159 * radius * radius;
    }
};

int main() {
    // Shape s; // Error: cannot instantiate abstract class

    Shape* s1 = new Rectangle(5, 4);
    Shape* s2 = new Circle(3);

    std::cout << "Area of rectangle: " << s1->getArea() << std::endl;
    std::cout << "Area of circle: " << s2->getArea() << std::endl;

    delete s1;
    delete s2;
    return 0;
}
```

## Virtual Destructors

It is important to make the destructor of a base class `virtual` if you are going to be deleting derived class objects through a base class pointer.

```cpp
class Base {
public:
    virtual ~Base() { ... }
};
```

If the destructor is not virtual, then `delete ptr` (where `ptr` is a `Base*` pointing to a `Derived` object) will only call the `Base` class destructor, leading to a memory leak if the `Derived` class has its own resources to clean up.
