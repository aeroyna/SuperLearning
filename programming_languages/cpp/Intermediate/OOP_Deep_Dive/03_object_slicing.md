# Object Slicing

**Object slicing** is a common pitfall in C++ where the derived part of an object is "sliced off" when it's copied or assigned to a base class object. This is a frequent interview topic.

## What Is Object Slicing?

When you assign a derived class object to a base class object (by value), only the base class portion is copied. The derived part is lost:

```cpp
class Base {
public:
    int baseData = 1;
    virtual void print() { std::cout << "Base: " << baseData << "\n"; }
};

class Derived : public Base {
public:
    int derivedData = 2;
    void print() override { std::cout << "Derived: " << baseData << ", " << derivedData << "\n"; }
};

int main() {
    Derived d;
    d.baseData = 10;
    d.derivedData = 20;

    Base b = d;  // SLICING! Only Base part is copied

    b.print();   // Calls Base::print(), prints "Base: 10"
                 // derivedData is LOST!

    return 0;
}
```

## Visual Representation

```
Derived object:           After slicing to Base:
┌─────────────────┐       ┌─────────────────┐
│ vptr → Derived  │       │ vptr → Base     │  ← Changed!
│ baseData = 10   │  ──►  │ baseData = 10   │
│ derivedData = 20│       └─────────────────┘
└─────────────────┘            ↑
                          derivedData LOST!
```

## Why Does It Happen?

When you copy to a base class object:
1. The base class's copy constructor/assignment is called
2. It only knows about base class members
3. The derived members aren't copied—they don't exist in the destination

```cpp
// This is essentially what happens:
Base b = d;
// Calls: Base::Base(const Base& other)
// Only copies baseData, not derivedData
```

## Common Scenarios Where Slicing Occurs

### 1. Assignment to Base Object

```cpp
Derived d;
Base b = d;  // Sliced
```

### 2. Passing by Value to Function

```cpp
void process(Base b) {  // Receives COPY, slicing occurs
    b.print();          // Always calls Base::print()
}

Derived d;
process(d);  // Sliced when passed!
```

### 3. Returning by Value

```cpp
Base getObject() {
    Derived d;
    return d;  // Sliced when returned
}
```

### 4. Storing in Container by Value

```cpp
std::vector<Base> objects;
objects.push_back(Derived());  // Sliced!

for (auto& obj : objects) {
    obj.print();  // Always Base::print()
}
```

## How to Prevent Object Slicing

### 1. Use Pointers

```cpp
void process(Base* b) {
    b->print();  // Virtual dispatch works correctly
}

Derived d;
process(&d);  // No slicing, d remains intact
```

### 2. Use References

```cpp
void process(Base& b) {
    b.print();  // Virtual dispatch works correctly
}

Derived d;
process(d);  // No slicing
```

### 3. Use Smart Pointers for Containers

```cpp
std::vector<std::unique_ptr<Base>> objects;
objects.push_back(std::make_unique<Derived>());

for (auto& obj : objects) {
    obj->print();  // Correctly calls Derived::print()
}
```

### 4. Make Base Class Abstract

If copying a base doesn't make sense, make it abstract:

```cpp
class Base {
public:
    virtual void print() = 0;  // Pure virtual
    virtual ~Base() = default;
};

// Now you CAN'T slice:
// Base b;  // Error: can't instantiate abstract class
```

### 5. Delete Copy Operations

```cpp
class Base {
public:
    Base() = default;
    Base(const Base&) = delete;             // No copy
    Base& operator=(const Base&) = delete;  // No assignment
    virtual ~Base() = default;
};
```

## Code Example: Slicing in Action

```cpp
#include <iostream>
#include <vector>
#include <memory>

class Shape {
public:
    virtual void draw() const { std::cout << "Drawing shape\n"; }
    virtual double area() const { return 0; }
    virtual ~Shape() = default;
};

class Circle : public Shape {
    double radius;
public:
    Circle(double r) : radius(r) {}
    void draw() const override { std::cout << "Drawing circle, r=" << radius << "\n"; }
    double area() const override { return 3.14159 * radius * radius; }
};

// BAD: Demonstrates slicing
void badExample() {
    std::cout << "=== Bad Example (Slicing) ===\n";

    std::vector<Shape> shapes;  // Stores by value - BAD!
    shapes.push_back(Circle(5));

    for (const auto& s : shapes) {
        s.draw();  // Calls Shape::draw(), not Circle::draw()!
        std::cout << "Area: " << s.area() << "\n";  // Returns 0!
    }
}

// GOOD: No slicing with pointers
void goodExample() {
    std::cout << "\n=== Good Example (No Slicing) ===\n";

    std::vector<std::unique_ptr<Shape>> shapes;
    shapes.push_back(std::make_unique<Circle>(5));

    for (const auto& s : shapes) {
        s->draw();  // Correctly calls Circle::draw()
        std::cout << "Area: " << s->area() << "\n";  // Correct area!
    }
}

int main() {
    badExample();
    goodExample();
    return 0;
}
```

Output:
```
=== Bad Example (Slicing) ===
Drawing shape
Area: 0

=== Good Example (No Slicing) ===
Drawing circle, r=5
Area: 78.5397
```

## Slicing with Assignment vs Construction

Both can cause slicing:

```cpp
Derived d;

// Construction - calls Base copy constructor
Base b1 = d;       // Slicing

// Assignment - calls Base assignment operator
Base b2;
b2 = d;            // Slicing

// Both lose the derived part
```

## Intentional Slicing

Sometimes slicing is intentional:

```cpp
class Employee {
public:
    std::string name;
    int id;
};

class Manager : public Employee {
public:
    std::vector<Employee*> reports;
};

// We might intentionally want just the Employee part
void printBasicInfo(Employee emp) {  // Intentional copy of base part
    std::cout << emp.name << " (" << emp.id << ")\n";
}
```

## Key Takeaways

- Object slicing occurs when derived objects are copied to base type by value
- The derived portion (members, vtable entry) is lost
- Prevents polymorphism from working correctly
- Solutions: use pointers, references, or smart pointers
- Consider making base classes abstract to prevent accidental slicing
- Containers should store pointers, not objects, for polymorphic types

## Common Interview Questions

> [!question]- What is object slicing?
> When a derived class object is copied to a base class object by value, losing the derived class's data members and virtual function overrides.

> [!question]- How do you prevent object slicing?
> Use pointers or references instead of values. Use smart pointers for containers. Make base classes abstract or delete copy operations.

> [!question]- Why doesn't slicing occur with pointers/references?
> Pointers and references maintain the connection to the original object. No copying occurs, so the derived part remains intact.

> [!question]- What happens to virtual functions after slicing?
> The vptr in the sliced copy points to the base class vtable, so virtual calls invoke base class versions.

## Related Topics

- [[../Object-Oriented_Programming/Polymorphism/02_virtual_functions|Virtual Functions]]
- [[05_shallow_vs_deep_copy|Shallow vs Deep Copy]]
- [[../Memory_Management/Smart_Pointers/01_smart_pointers_intro|Smart Pointers]]
