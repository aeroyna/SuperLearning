# Increment and Decrement Operators

The increment (`++`) and decrement (`--`) operators come in two forms: **prefix** and **postfix**. Both can be overloaded, but they have different signatures and semantics.

## Prefix vs Postfix

| Operator | Syntax | Returns | Efficiency |
|----------|--------|---------|------------|
| Prefix `++` | `++obj` | Reference to modified object | More efficient |
| Postfix `++` | `obj++` | Copy of original value | Less efficient |

## Basic Implementation

```cpp
#include <iostream>

class Counter {
private:
    int value;

public:
    Counter(int v = 0) : value(v) {}

    // Prefix increment: ++obj
    Counter& operator++() {
        ++value;
        return *this;  // Return reference to modified object
    }

    // Postfix increment: obj++
    // The dummy int parameter distinguishes it from prefix
    Counter operator++(int) {
        Counter temp = *this;  // Save current state
        ++value;               // Increment
        return temp;           // Return OLD value
    }

    // Prefix decrement: --obj
    Counter& operator--() {
        --value;
        return *this;
    }

    // Postfix decrement: obj--
    Counter operator--(int) {
        Counter temp = *this;
        --value;
        return temp;
    }

    int getValue() const { return value; }
};

int main() {
    Counter c(5);

    std::cout << "Initial: " << c.getValue() << std::endl;     // 5

    std::cout << "++c: " << (++c).getValue() << std::endl;     // 6
    std::cout << "After: " << c.getValue() << std::endl;       // 6

    std::cout << "c++: " << (c++).getValue() << std::endl;     // 6 (old value)
    std::cout << "After: " << c.getValue() << std::endl;       // 7

    return 0;
}
```

## Why the Dummy `int` Parameter?

The compiler needs to distinguish between prefix and postfix:

```cpp
Counter& operator++();      // Prefix: ++obj
Counter operator++(int);    // Postfix: obj++ (int is dummy, never used)
```

When you write `obj++`, the compiler calls `obj.operator++(0)` (with a dummy 0).

## Efficiency Considerations

```cpp
// Prefix: modify and return reference (efficient)
Counter& operator++() {
    ++value;
    return *this;  // No copy needed
}

// Postfix: must make a copy (less efficient)
Counter operator++(int) {
    Counter temp = *this;  // COPY created
    ++value;
    return temp;  // Another copy might occur
}
```

> [!tip] Best Practice
> Prefer prefix `++c` over postfix `c++` when you don't need the old value. For built-in types, the compiler optimizes this away, but for complex objects, prefix is faster.

## Implementing Postfix Using Prefix

To avoid code duplication:

```cpp
class Counter {
public:
    // Prefix: the "real" implementation
    Counter& operator++() {
        ++value;
        return *this;
    }

    // Postfix: uses prefix
    Counter operator++(int) {
        Counter temp = *this;
        ++(*this);  // Call prefix operator
        return temp;
    }
};
```

## Iterator Example

Iterators heavily use increment operators:

```cpp
#include <iostream>

class IntRange {
public:
    class Iterator {
    private:
        int current;

    public:
        Iterator(int val) : current(val) {}

        int operator*() const { return current; }

        // Prefix increment
        Iterator& operator++() {
            ++current;
            return *this;
        }

        // Postfix increment
        Iterator operator++(int) {
            Iterator temp = *this;
            ++(*this);
            return temp;
        }

        bool operator!=(const Iterator& other) const {
            return current != other.current;
        }
    };

    IntRange(int start, int end) : start_(start), end_(end) {}

    Iterator begin() { return Iterator(start_); }
    Iterator end() { return Iterator(end_); }

private:
    int start_, end_;
};

int main() {
    for (int x : IntRange(1, 5)) {
        std::cout << x << " ";  // 1 2 3 4
    }
    std::cout << std::endl;

    return 0;
}
```

## Chaining Increment Operators

Prefix returns a reference, allowing chaining:

```cpp
Counter c(0);
++++c;  // Equivalent to ++(++c), c becomes 2

// Postfix cannot be chained meaningfully:
// c++++; // Returns temporary, second ++ operates on temporary
```

## Smart Pointer Example

```cpp
template<typename T>
class ArrayPtr {
private:
    T* ptr;

public:
    ArrayPtr(T* p) : ptr(p) {}

    T& operator*() { return *ptr; }
    T* operator->() { return ptr; }

    // Move to next element
    ArrayPtr& operator++() {
        ++ptr;
        return *this;
    }

    ArrayPtr operator++(int) {
        ArrayPtr temp = *this;
        ++ptr;
        return temp;
    }

    ArrayPtr& operator--() {
        --ptr;
        return *this;
    }
};

int main() {
    int arr[] = {10, 20, 30, 40};
    ArrayPtr<int> p(arr);

    std::cout << *p << std::endl;    // 10
    ++p;
    std::cout << *p << std::endl;    // 20
    std::cout << *p++ << std::endl;  // 20 (postfix returns old)
    std::cout << *p << std::endl;    // 30

    return 0;
}
```

## Key Takeaways

- Prefix `++obj` returns reference, modifies first
- Postfix `obj++` returns copy of old value, then modifies
- Use dummy `int` parameter to distinguish postfix
- Prefix is more efficient (no copy needed)
- Implement postfix using prefix to avoid duplication
- Essential for custom iterators

## Common Interview Questions

> [!question]- How does the compiler differentiate prefix from postfix?
> Prefix: `operator++()` with no parameters. Postfix: `operator++(int)` with a dummy int parameter that's never used.

> [!question]- Why is prefix more efficient?
> Prefix modifies in place and returns a reference. Postfix must create a copy of the original value to return, then modify the object.

> [!question]- What does `++++c` do?
> It increments `c` twice. First `++c` increments and returns reference to `c`, then the outer `++` increments again.

## Related Topics

- [[../Standard_Template_Library/Iterators/01_iterators|Iterators]]
- [[01_arithmetic_operators|Arithmetic Operators]]
