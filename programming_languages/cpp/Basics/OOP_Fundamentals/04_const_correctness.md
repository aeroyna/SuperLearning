# Const Correctness

Const correctness is a fundamental C++ concept that helps prevent bugs and communicates intent. It's a common interview topic.

## What Is Const Correctness?

Using `const` wherever possible to indicate that something won't be modified. This:
1. Catches bugs at compile time
2. Documents intent
3. Enables compiler optimizations

## Const Variables

```cpp
const int MAX_SIZE = 100;  // Cannot be modified
MAX_SIZE = 200;  // Error!

int value = 42;
const int& ref = value;  // Can read through ref, not modify
ref = 50;  // Error!
```

## Const with Pointers

The **two-const rule** - const can apply to:
1. What the pointer points TO (the data)
2. The pointer ITSELF

```cpp
int x = 10, y = 20;

// 1. Pointer to const int (data is const)
const int* p1 = &x;      // Can't modify *p1
// *p1 = 50;             // Error!
p1 = &y;                 // OK - pointer itself can change

// 2. Const pointer to int (pointer is const)
int* const p2 = &x;      // Can't modify p2
*p2 = 50;                // OK - can modify data
// p2 = &y;              // Error!

// 3. Const pointer to const int (both const)
const int* const p3 = &x;
// *p3 = 50;             // Error!
// p3 = &y;              // Error!
```

### Reading Pointer Declarations

Read right-to-left:
```cpp
const int* p;      // p is a pointer to int that is const
int const* p;      // Same as above (const int* == int const*)
int* const p;      // p is a const pointer to int
const int* const p; // p is a const pointer to const int
```

## Const Member Functions

A const member function promises not to modify the object:

```cpp
class Rectangle {
    int width, height;

public:
    Rectangle(int w, int h) : width(w), height(h) {}

    // Const member function - can be called on const objects
    int area() const {
        return width * height;
        // width = 10;  // Error! Can't modify members
    }

    // Non-const - can only be called on non-const objects
    void resize(int w, int h) {
        width = w;
        height = h;
    }
};

void process(const Rectangle& r) {
    r.area();     // OK - area() is const
    // r.resize(5, 5);  // Error! resize() is not const
}
```

### What Const Member Functions CAN'T Do

```cpp
class Example {
    int value;
    int* ptr;

public:
    void constMethod() const {
        // value = 10;      // Error! Can't modify member
        // ptr = nullptr;   // Error! Can't modify ptr
        *ptr = 10;          // OK! ptr is const, but *ptr isn't
    }
};
```

> [!warning] Logical vs Bitwise Const
> `const` is **bitwise** const by default. The pointer `ptr` can't change, but what it points to can. Use `mutable` for logical const.

## The mutable Keyword

For members that can change even in const methods (like caches):

```cpp
class ExpensiveCalculator {
    mutable int cachedResult;
    mutable bool cacheValid = false;

public:
    int compute(int x) const {  // logically const
        if (!cacheValid) {
            cachedResult = /* expensive computation */;
            cacheValid = true;
        }
        return cachedResult;
    }
};
```

## Const Overloading

Provide both const and non-const versions:

```cpp
class Array {
    int* data;

public:
    // Non-const version - returns modifiable reference
    int& operator[](size_t i) {
        return data[i];
    }

    // Const version - returns const reference
    const int& operator[](size_t i) const {
        return data[i];
    }
};

void example() {
    Array arr;
    arr[0] = 10;           // Calls non-const version

    const Array& cref = arr;
    int x = cref[0];       // Calls const version
    // cref[0] = 5;        // Error! Returns const reference
}
```

### Avoiding Duplication

```cpp
class Array {
    int* data;

public:
    const int& operator[](size_t i) const {
        // ... bounds checking, etc.
        return data[i];
    }

    int& operator[](size_t i) {
        // Reuse const version
        return const_cast<int&>(
            static_cast<const Array&>(*this)[i]
        );
    }
};
```

## Const Return Types

### Returning Const Reference

```cpp
class Person {
    std::string name;

public:
    // Return const reference to prevent modification
    const std::string& getName() const {
        return name;
    }
};

Person p;
// p.getName() = "New";  // Error! Can't modify through const ref
std::string copy = p.getName();  // OK - makes a copy
```

### Returning by Value

For return by value, `const` on primitives is useless:

```cpp
const int getValue() { return 42; }  // Const is meaningless here
int x = getValue();  // x is a copy anyway
```

But for objects, it prevents calling non-const methods on temporaries:

```cpp
const std::string getString() { return "hello"; }
getString().append(" world");  // Error! append() is non-const
```

## Const Parameters

```cpp
// Pass by const reference - most common for objects
void process(const std::string& str) {
    // Can read str, can't modify
}

// Pass by const pointer
void process(const int* arr, size_t size) {
    // Can read arr[i], can't modify
}

// Pass by value - const is redundant (it's a copy)
void process(const int x) {  // const is internal implementation detail
    // Doesn't affect caller
}
```

## Const with STL

```cpp
std::vector<int> vec = {1, 2, 3, 4, 5};

// Non-const iterator - can modify
for (auto it = vec.begin(); it != vec.end(); ++it) {
    *it = *it * 2;  // OK
}

// Const iterator - read-only
for (auto it = vec.cbegin(); it != vec.cend(); ++it) {
    // *it = 0;  // Error!
}

// Range-based with const
for (const auto& elem : vec) {
    // elem = 0;  // Error!
}
```

## Best Practices

1. **Make member functions const if they don't modify state**
2. **Use const references for function parameters (objects)**
3. **Return const references to prevent modification of members**
4. **Prefer `const_iterator` when not modifying**
5. **Use `mutable` for logical const (caches)**

## Key Takeaways

- Const correctness prevents bugs and documents intent
- Const member functions can be called on const objects
- Provide both const and non-const overloads when needed
- `mutable` allows modification in const member functions
- Read pointer declarations right-to-left

## Common Interview Questions

> [!question]- What's the difference between `const int*` and `int* const`?
> `const int*` - pointer to const int (can't modify data). `int* const` - const pointer to int (can't modify pointer).

> [!question]- Can a const member function modify members?
> Not directly. But it can modify `mutable` members and data pointed to by pointers (the pointer itself is const, not the pointee).

> [!question]- Why use const references for parameters?
> Avoids copying (efficient), prevents modification (safe), and allows passing both lvalues and rvalues.

> [!question]- What is logical vs bitwise const?
> Bitwise: no bits of the object change. Logical: the object appears unchanged to users (internal caches may update). `mutable` supports logical const.

## Related Topics

- [[../../Intermediate/Type_Casting/03_const_cast|const_cast]]
- [[../../Intermediate/Operator_Overloading/05_subscript_and_function_call|Subscript Operator]]
