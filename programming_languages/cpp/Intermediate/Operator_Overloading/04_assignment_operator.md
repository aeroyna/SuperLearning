# Assignment Operator Overloading

The assignment operator (`=`) is one of the most important operators to understand, especially when your class manages resources like dynamic memory.

## Default Assignment Operator

If you don't define an assignment operator, the compiler generates one that performs **member-wise copy** (shallow copy).

```cpp
class Simple {
public:
    int x;
    double y;
};

Simple a{1, 2.0};
Simple b;
b = a;  // Compiler-generated: b.x = a.x; b.y = a.y;
```

## When to Define Your Own

You need a custom assignment operator when your class:
- Manages dynamic memory (raw pointers)
- Holds file handles or other resources
- Contains unique ownership semantics

## The Problem: Shallow Copy

```cpp
class BadString {
private:
    char* data;
    size_t size;

public:
    BadString(const char* s = "") {
        size = strlen(s);
        data = new char[size + 1];
        strcpy(data, s);
    }

    ~BadString() {
        delete[] data;  // Free memory
    }
};

int main() {
    BadString s1("Hello");
    BadString s2("World");
    s2 = s1;  // DANGER! Default shallow copy

    // Now both s1.data and s2.data point to same memory!
    // When s1 and s2 are destroyed, double-free occurs!
}
```

```
Memory visualization:

Before assignment:
s1.data ──→ [ H | e | l | l | o | \0 ]
s2.data ──→ [ W | o | r | l | d | \0 ]

After shallow copy (s2 = s1):
s1.data ──→ [ H | e | l | l | o | \0 ]
s2.data ──↗
           [ W | o | r | l | d | \0 ]  ← LEAKED!
```

## Correct Implementation: Deep Copy

```cpp
#include <cstring>
#include <algorithm>

class String {
private:
    char* data;
    size_t size;

public:
    // Constructor
    String(const char* s = "") {
        size = strlen(s);
        data = new char[size + 1];
        strcpy(data, s);
    }

    // Copy Constructor
    String(const String& other) {
        size = other.size;
        data = new char[size + 1];
        strcpy(data, other.data);
    }

    // Destructor
    ~String() {
        delete[] data;
    }

    // Copy Assignment Operator
    String& operator=(const String& other) {
        // 1. Check for self-assignment
        if (this == &other) {
            return *this;
        }

        // 2. Free existing resource
        delete[] data;

        // 3. Allocate new resource and copy
        size = other.size;
        data = new char[size + 1];
        strcpy(data, other.data);

        // 4. Return *this for chaining
        return *this;
    }

    void print() const {
        std::cout << data << std::endl;
    }
};

int main() {
    String s1("Hello");
    String s2("World");

    s2 = s1;  // Deep copy - safe!

    s1.print();  // Hello
    s2.print();  // Hello (independent copy)

    return 0;
}
```

## Why Check for Self-Assignment?

```cpp
String s("Hello");
s = s;  // Self-assignment

// Without the check:
// 1. delete[] data; (we just deleted our own data!)
// 2. data = new char[size + 1];
// 3. strcpy(data, other.data); (copying from deleted memory - UB!)
```

## Copy-and-Swap Idiom (Recommended)

A safer, exception-safe implementation:

```cpp
class String {
private:
    char* data;
    size_t size;

public:
    // ... constructor, copy constructor, destructor ...

    // Swap function
    void swap(String& other) noexcept {
        std::swap(data, other.data);
        std::swap(size, other.size);
    }

    // Copy Assignment using copy-and-swap
    String& operator=(String other) {  // Note: pass by VALUE (creates copy)
        swap(other);  // Swap with the copy
        return *this;
    }  // 'other' (old data) destroyed here
};
```

### Why Copy-and-Swap is Better

1. **Exception-safe**: If copy constructor throws, original object unchanged
2. **Self-assignment safe**: Works correctly without explicit check
3. **Code reuse**: Uses copy constructor, reducing duplication
4. **Simple**: Harder to get wrong

## Move Assignment Operator (C++11)

For efficient assignment from temporaries:

```cpp
class String {
public:
    // Move Assignment Operator
    String& operator=(String&& other) noexcept {
        if (this != &other) {
            delete[] data;          // Free existing
            data = other.data;      // Steal resource
            size = other.size;
            other.data = nullptr;   // Leave source valid
            other.size = 0;
        }
        return *this;
    }
};

String createString() {
    return String("Temporary");
}

int main() {
    String s;
    s = createString();  // Move assignment (no copy!)
}
```

## Combined Copy-and-Swap with Move

```cpp
class String {
public:
    // Single assignment operator handles both copy and move!
    String& operator=(String other) noexcept {  // Pass by value
        swap(other);
        return *this;
    }
    // - If passed lvalue: copy constructor called, then swap
    // - If passed rvalue: move constructor called, then swap
};
```

## The Rule of Three/Five/Zero

| Rule | Special Members to Define |
|------|---------------------------|
| **Rule of Three** | Destructor, Copy Constructor, Copy Assignment |
| **Rule of Five** | + Move Constructor, Move Assignment |
| **Rule of Zero** | None (use smart pointers/RAII) |

> [!tip] Modern C++ Recommendation
> Prefer Rule of Zero by using `std::string`, `std::vector`, and smart pointers. Let the compiler generate correct defaults.

```cpp
// Rule of Zero - all special members auto-generated correctly
class ModernPerson {
private:
    std::string name;           // Handles its own memory
    std::vector<int> scores;    // Handles its own memory
};
```

## Key Takeaways

- Default assignment does shallow copy (member-wise)
- Define custom `operator=` for classes managing resources
- Always check for self-assignment (or use copy-and-swap)
- Return `*this` by reference for chaining (`a = b = c`)
- Copy-and-swap idiom provides exception safety
- Mark move assignment as `noexcept` for optimization
- Prefer Rule of Zero with RAII types

## Common Interview Questions

> [!question]- What happens if you don't handle self-assignment?
> For classes with dynamic resources, self-assignment without a check can delete the object's data before copying from it, causing undefined behavior.

> [!question]- Difference between copy constructor and copy assignment?
> Copy constructor initializes a NEW object from an existing one. Copy assignment replaces an EXISTING object's data with a copy from another.

> [!question]- Why return `*this` by reference?
> To enable chaining: `a = b = c`. Also, returning by value would create an unnecessary copy.

## Related Topics

- [[../OOP_Deep_Dive/05_shallow_vs_deep_copy|Shallow vs Deep Copy]]
- [[../../Advanced/Move_Semantics_and_Rvalue_References/03_move_constructors_and_move_assignment|Move Semantics]]
- [[../Memory_Management/RAII/01_raii|RAII]]
