# Practice Problems: Memory Management

## Problem 1: Convert a raw pointer to a smart pointer

You have a function that returns a raw pointer to a dynamically allocated object. Modify the code to use a `std::unique_ptr` to manage the object's lifetime.

```cpp
#include <iostream>

struct MyStruct {
    MyStruct() { std::cout << "MyStruct created\n"; }
    ~MyStruct() { std::cout << "MyStruct destroyed\n"; }
    void do_something() { std::cout << "Doing something...\n"; }
};

MyStruct* create_raw_pointer() {
    return new MyStruct();
}

int main() {
    MyStruct* raw_ptr = create_raw_pointer();
    raw_ptr->do_something();
    delete raw_ptr; // manual deletion
    return 0;
}
```

### Solution

```cpp
#include <iostream>
#include <memory>

struct MyStruct {
    MyStruct() { std::cout << "MyStruct created\n"; }
    ~MyStruct() { std::cout << "MyStruct destroyed\n"; }
    void do_something() { std::cout << "Doing something...\n"; }
};

std::unique_ptr<MyStruct> create_unique_pointer() {
    return std::make_unique<MyStruct>();
}

int main() {
    std::unique_ptr<MyStruct> smart_ptr = create_unique_pointer();
    smart_ptr->do_something();
    // No need to call delete, the object is automatically destroyed
    return 0;
}
```

## Problem 2: `std::shared_ptr` in a container

You have a `std::vector` that needs to store pointers to a base class `Shape`. Some of these shapes are shared between different parts of your application. Use `std::shared_ptr` to manage the shapes in the vector.

### Solution

```cpp
#include <iostream>
#include <vector>
#include <memory>

class Shape {
public:
    virtual void draw() = 0;
    virtual ~Shape() = default;
};

class Circle : public Shape {
public:
    void draw() override { std::cout << "Drawing a circle\n"; }
};

class Rectangle : public Shape {
public:
    void draw() override { std::cout << "Drawing a rectangle\n"; }
};

int main() {
    // A vector to hold shared pointers to shapes
    std::vector<std::shared_ptr<Shape>> shapes;

    // Create some shapes
    auto circle1 = std::make_shared<Circle>();
    auto rect1 = std::make_shared<Rectangle>();

    // Add them to the vector
    shapes.push_back(circle1);
    shapes.push_back(rect1);
    shapes.push_back(circle1); // Add the same circle again

    // Another part of the application also holds a reference to circle1
    std::shared_ptr<Shape> another_ref_to_circle = circle1;

    std::cout << "Circle1 use count: " << circle1.use_count() << std::endl; // Should be 3

    // Draw all the shapes
    for (const auto& shape : shapes) {
        shape->draw();
    }

    return 0; // All memory is automatically deallocated
}
```

## Problem 3: Fix a circular reference

The following code has a memory leak due to a circular reference. Identify the problem and fix it using `std::weak_ptr`.

```cpp
#include <iostream>
#include <memory>

struct Person {
    std::string name;
    std::shared_ptr<Person> partner;
    Person(std::string n) : name(n) { std::cout << name << " created\n"; }
    ~Person() { std::cout << name << " destroyed\n"; }
};

int main() {
    auto lucy = std::make_shared<Person>("Lucy");
    auto ricky = std::make_shared<Person>("Ricky");

    lucy->partner = ricky;
    ricky->partner = lucy;

    return 0; // Destructors are not called
}
```

### Solution

```cpp
#include <iostream>
#include <memory>
#include <string>

struct Person {
    std::string name;
    std::weak_ptr<Person> partner; // Use weak_ptr to break the cycle
    Person(std::string n) : name(n) { std::cout << name << " created\n"; }
    ~Person() { std::cout << name << " destroyed\n"; }
};

int main() {
    auto lucy = std::make_shared<Person>("Lucy");
    auto ricky = std::make_shared<Person>("Ricky");

    lucy->partner = ricky;
    ricky->partner = lucy;
    
    // To access the partner, you would use lock()
    if (auto ricky_partner = ricky->partner.lock()) {
        std::cout << "Ricky's partner is " << ricky_partner->name << std::endl;
    }

    return 0; // Destructors are now called correctly
}
```
