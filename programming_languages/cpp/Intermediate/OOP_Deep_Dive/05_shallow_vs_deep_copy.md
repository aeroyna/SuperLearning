# Shallow Copy vs Deep Copy

Understanding the difference between shallow and deep copying is essential for managing resources in C++ classes.

## What Is Shallow Copy?

A **shallow copy** copies the values of all members directly, including pointers. Two objects end up pointing to the **same memory**.

```cpp
class Shallow {
public:
    int* data;

    Shallow(int val) {
        data = new int(val);
    }

    // Default copy constructor does shallow copy
    // Shallow(const Shallow& other) : data(other.data) { }
};

Shallow a(42);
Shallow b = a;  // Shallow copy!

// Both a.data and b.data point to the SAME memory
*b.data = 100;
std::cout << *a.data;  // Also 100!
```

### Memory Visualization

```
After shallow copy (Shallow b = a):

a.data ───┐
          ├──→ [ 42 ]  ← Same memory!
b.data ───┘
```

### Problems with Shallow Copy

1. **Double free**: When `a` and `b` are destroyed, both try to `delete` the same memory
2. **Unintended sharing**: Modifications through one object affect the other
3. **Dangling pointers**: If one object deletes the memory, the other has a dangling pointer

```cpp
{
    Shallow a(42);
    Shallow b = a;  // Shallow copy
}  // CRASH! Both a and b try to delete same memory
```

## What Is Deep Copy?

A **deep copy** allocates new memory and copies the **contents** of the pointed-to data.

```cpp
class Deep {
public:
    int* data;

    Deep(int val) {
        data = new int(val);
    }

    // Deep copy constructor
    Deep(const Deep& other) {
        data = new int(*other.data);  // Allocate NEW memory, copy value
    }

    // Deep copy assignment
    Deep& operator=(const Deep& other) {
        if (this != &other) {
            delete data;  // Free old memory
            data = new int(*other.data);  // Allocate new, copy value
        }
        return *this;
    }

    ~Deep() {
        delete data;
    }
};

Deep a(42);
Deep b = a;  // Deep copy!

*b.data = 100;
std::cout << *a.data;  // Still 42 - independent copies!
```

### Memory Visualization

```
After deep copy (Deep b = a):

a.data ───→ [ 42 ]   ← Separate memory

b.data ───→ [ 42 ]   ← Independent copy
```

## Comparison

| Aspect | Shallow Copy | Deep Copy |
|--------|-------------|-----------|
| What's copied | Pointer values | Actual data |
| Memory usage | Less (shared) | More (duplicated) |
| Independence | Objects share data | Objects are independent |
| Default behavior | Yes (compiler-generated) | No (must implement) |
| Performance | Fast | Slower (allocations) |
| Safety | Dangerous with pointers | Safe |

## Complete Example

```cpp
#include <iostream>
#include <cstring>

class String {
private:
    char* data;
    size_t length;

public:
    // Constructor
    String(const char* s = "") {
        length = strlen(s);
        data = new char[length + 1];
        strcpy(data, s);
        std::cout << "Constructor: " << data << "\n";
    }

    // Deep Copy Constructor
    String(const String& other) {
        length = other.length;
        data = new char[length + 1];  // NEW allocation
        strcpy(data, other.data);     // Copy contents
        std::cout << "Copy Constructor (deep): " << data << "\n";
    }

    // Deep Copy Assignment
    String& operator=(const String& other) {
        std::cout << "Copy Assignment (deep)\n";

        if (this != &other) {
            delete[] data;            // Free old memory

            length = other.length;
            data = new char[length + 1];
            strcpy(data, other.data);
        }
        return *this;
    }

    // Destructor
    ~String() {
        std::cout << "Destructor: " << (data ? data : "null") << "\n";
        delete[] data;
    }

    void print() const {
        std::cout << "String: " << data << "\n";
    }

    void modify(const char* s) {
        delete[] data;
        length = strlen(s);
        data = new char[length + 1];
        strcpy(data, s);
    }
};

int main() {
    String s1("Hello");
    String s2 = s1;     // Copy constructor

    s2.modify("World");

    s1.print();  // "Hello" - unaffected
    s2.print();  // "World" - modified

    String s3("Temp");
    s3 = s1;     // Copy assignment

    return 0;
}
```

## When Shallow Copy Is OK

Shallow copy is fine when:
- Class has no pointers/resources
- Class uses smart pointers that handle copying
- You actually want shared ownership (rare)

```cpp
// OK - no pointers
class Point {
    int x, y;
};

// OK - smart pointers handle it
class Modern {
    std::shared_ptr<int> data;
    std::vector<int> values;
};
```

## The Rule of Three/Five/Zero

### Rule of Three (C++98)

If you define any of these, you probably need all three:
1. Destructor
2. Copy constructor
3. Copy assignment operator

### Rule of Five (C++11)

Add move operations:
4. Move constructor
5. Move assignment operator

### Rule of Zero (Modern C++)

Best approach: Use RAII types (smart pointers, containers) and let the compiler generate all special members.

```cpp
// Rule of Zero - compiler handles everything correctly
class Modern {
    std::string name;
    std::vector<int> data;
    std::unique_ptr<Resource> resource;
    // No destructor, copy/move constructors/assignments needed!
};
```

## Deep Copy with Arrays

```cpp
class IntArray {
    int* arr;
    size_t size;

public:
    IntArray(size_t n) : size(n), arr(new int[n]()) {}

    // Deep copy constructor
    IntArray(const IntArray& other) : size(other.size) {
        arr = new int[size];
        std::copy(other.arr, other.arr + size, arr);
    }

    // Deep copy assignment
    IntArray& operator=(const IntArray& other) {
        if (this != &other) {
            delete[] arr;

            size = other.size;
            arr = new int[size];
            std::copy(other.arr, other.arr + size, arr);
        }
        return *this;
    }

    ~IntArray() { delete[] arr; }
};
```

## Copy-and-Swap Idiom (Recommended)

A safer way to implement copy assignment:

```cpp
class String {
    char* data;
    size_t length;

public:
    // ... constructor, copy constructor, destructor ...

    // Swap function
    void swap(String& other) noexcept {
        std::swap(data, other.data);
        std::swap(length, other.length);
    }

    // Copy-and-swap assignment
    String& operator=(String other) {  // Note: pass by value
        swap(other);
        return *this;
    }  // 'other' destroyed here, cleaning up old data
};
```

Benefits:
- Exception safe
- Handles self-assignment
- Reuses copy constructor

## Key Takeaways

- Shallow copy: copies pointer values (shared memory)
- Deep copy: allocates new memory and copies contents
- Default copy is shallow—dangerous with raw pointers
- Follow Rule of Three/Five when managing resources
- Prefer Rule of Zero with RAII types
- Copy-and-swap provides exception safety

## Common Interview Questions

> [!question]- What's the difference between shallow and deep copy?
> Shallow copies pointer values (both point to same memory). Deep allocates new memory and copies the pointed-to data.

> [!question]- When do you need to implement deep copy?
> When your class manages resources through raw pointers and you want objects to be independent.

> [!question]- What problems occur with shallow copy of objects containing pointers?
> Double-free (both destructors delete same memory), unintended data sharing, and dangling pointers.

> [!question]- What's the Rule of Three?
> If you define a destructor, copy constructor, or copy assignment operator, you should probably define all three.

## Related Topics

- [[../Operator_Overloading/04_assignment_operator|Assignment Operator]]
- [[../../Advanced/Move_Semantics_and_Rvalue_References/03_move_constructors_and_move_assignment|Move Semantics]]
- [[../Memory_Management/Smart_Pointers/01_smart_pointers_intro|Smart Pointers]]
