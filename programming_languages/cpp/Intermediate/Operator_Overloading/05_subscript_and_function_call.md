# Subscript and Function Call Operators

The subscript operator `[]` and function call operator `()` are powerful tools that allow your objects to behave like arrays and functions respectively.

## Subscript Operator (`[]`)

The subscript operator allows array-like access to elements in your class.

### Basic Example: Array Wrapper

```cpp
#include <iostream>
#include <stdexcept>

class IntArray {
private:
    int* data;
    size_t size;

public:
    IntArray(size_t n) : size(n) {
        data = new int[n]();
    }

    ~IntArray() {
        delete[] data;
    }

    // Non-const version (for modification)
    int& operator[](size_t index) {
        if (index >= size) {
            throw std::out_of_range("Index out of bounds");
        }
        return data[index];
    }

    // Const version (for read-only access)
    const int& operator[](size_t index) const {
        if (index >= size) {
            throw std::out_of_range("Index out of bounds");
        }
        return data[index];
    }

    size_t getSize() const { return size; }
};

int main() {
    IntArray arr(5);

    arr[0] = 10;      // Uses non-const operator[]
    arr[1] = 20;

    std::cout << arr[0] << std::endl;  // 10

    const IntArray& constRef = arr;
    std::cout << constRef[0] << std::endl;  // Uses const operator[]
    // constRef[0] = 5;  // Error! Can't modify through const

    return 0;
}
```

### Why Two Versions?

```cpp
// Non-const: returns int& (modifiable reference)
int& operator[](size_t index);

// Const: returns const int& (read-only reference)
const int& operator[](size_t index) const;
```

| Scenario | Version Called |
|----------|----------------|
| `arr[i] = 5;` | Non-const (modification) |
| `int x = arr[i];` | Either (depends on context) |
| `const IntArray& r = arr; r[i];` | Const only |

### Multi-dimensional Access

For 2D arrays, you can either:

1. **Return a proxy/row object:**
```cpp
class Matrix {
    int* data;
    size_t rows, cols;

public:
    class RowProxy {
        int* row;
    public:
        RowProxy(int* r) : row(r) {}
        int& operator[](size_t col) { return row[col]; }
    };

    RowProxy operator[](size_t row) {
        return RowProxy(data + row * cols);
    }
};

Matrix m(3, 4);
m[1][2] = 5;  // m.operator[](1).operator[](2)
```

2. **Use `operator()` instead** (more common):
```cpp
class Matrix {
public:
    int& operator()(size_t row, size_t col) {
        return data[row * cols + col];
    }
};

Matrix m(3, 4);
m(1, 2) = 5;  // Cleaner for multi-dimensional
```

## Function Call Operator (`()`) - Functors

The function call operator makes objects callable like functions. Objects with `operator()` are called **functors** or **function objects**.

### Basic Functor

```cpp
#include <iostream>

class Multiplier {
private:
    int factor;

public:
    Multiplier(int f) : factor(f) {}

    int operator()(int x) const {
        return x * factor;
    }
};

int main() {
    Multiplier times3(3);
    Multiplier times5(5);

    std::cout << times3(10) << std::endl;  // 30
    std::cout << times5(10) << std::endl;  // 50

    return 0;
}
```

### Functors vs Functions

| Feature | Function | Functor |
|---------|----------|---------|
| State | No (unless static/global) | Yes (member variables) |
| Inline potential | Maybe | Better (compiler knows type) |
| Customization | New function needed | Change constructor args |

### Functors with STL Algorithms

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

class IsGreaterThan {
private:
    int threshold;

public:
    IsGreaterThan(int t) : threshold(t) {}

    bool operator()(int x) const {
        return x > threshold;
    }
};

int main() {
    std::vector<int> nums = {1, 5, 3, 8, 2, 9, 4};

    // Count numbers greater than 4
    int count = std::count_if(nums.begin(), nums.end(), IsGreaterThan(4));
    std::cout << "Count > 4: " << count << std::endl;  // 3

    // Using lambda (modern alternative)
    int count2 = std::count_if(nums.begin(), nums.end(),
                               [](int x) { return x > 4; });

    return 0;
}
```

### Stateful Functors

```cpp
class Counter {
private:
    mutable int count = 0;  // mutable allows modification in const method

public:
    void operator()(int x) const {
        ++count;
        std::cout << "Call " << count << ": " << x << std::endl;
    }

    int getCount() const { return count; }
};

int main() {
    Counter c;
    c(10);  // Call 1: 10
    c(20);  // Call 2: 20
    c(30);  // Call 3: 30

    std::cout << "Total calls: " << c.getCount() << std::endl;  // 3
}
```

### Generic Functors

```cpp
template<typename T>
class Adder {
private:
    T value;

public:
    Adder(T v) : value(v) {}

    T operator()(T x) const {
        return x + value;
    }
};

int main() {
    Adder<int> add5(5);
    Adder<double> add3_14(3.14);

    std::cout << add5(10) << std::endl;      // 15
    std::cout << add3_14(1.0) << std::endl;  // 4.14
}
```

## Standard Library Functors

C++ provides built-in functors in `<functional>`:

```cpp
#include <functional>
#include <algorithm>
#include <vector>

std::vector<int> nums = {3, 1, 4, 1, 5};

// Sort descending using std::greater
std::sort(nums.begin(), nums.end(), std::greater<int>());

// Other useful functors:
std::plus<int>()      // a + b
std::minus<int>()     // a - b
std::multiplies<int>() // a * b
std::divides<int>()   // a / b
std::less<int>()      // a < b
std::equal_to<int>()  // a == b
```

## Key Takeaways

- `operator[]` must be a member function
- Provide both const and non-const versions of `operator[]`
- Use `operator()` for multi-dimensional indexing or callable objects
- Functors can hold state, unlike regular functions
- Functors are heavily used with STL algorithms
- Lambdas are often a cleaner modern alternative to functors

## Common Interview Questions

> [!question]- Why provide two versions of `operator[]`?
> The const version allows read access through const references/pointers. The non-const version allows modification. The compiler selects based on the constness of the object.

> [!question]- What's the advantage of functors over function pointers?
> Functors can hold state, are easier to inline (the compiler knows the exact type), and can be customized through constructor parameters.

> [!question]- Can `operator[]` take multiple arguments?
> In C++23, yes (multidimensional subscript). Before that, you need `operator()` or return a proxy object.

## Related Topics

- [[../../Advanced/Lambda_Expressions/01_lambda_expressions|Lambda Expressions]]
- [[../Standard_Template_Library/Algorithms/01_algorithms|STL Algorithms]]
