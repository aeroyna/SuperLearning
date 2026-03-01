# Arithmetic Operator Overloading

Arithmetic operators (`+`, `-`, `*`, `/`, `%`) are commonly overloaded for mathematical classes like vectors, matrices, and complex numbers.

## Basic Example: Complex Numbers

```cpp
#include <iostream>

class Complex {
private:
    double real, imag;

public:
    Complex(double r = 0, double i = 0) : real(r), imag(i) {}

    // Overload + as a member function
    Complex operator+(const Complex& other) const {
        return Complex(real + other.real, imag + other.imag);
    }

    // Overload - as a member function
    Complex operator-(const Complex& other) const {
        return Complex(real - other.real, imag - other.imag);
    }

    // Overload * as a member function
    Complex operator*(const Complex& other) const {
        return Complex(
            real * other.real - imag * other.imag,
            real * other.imag + imag * other.real
        );
    }

    // Unary minus (negation)
    Complex operator-() const {
        return Complex(-real, -imag);
    }

    void print() const {
        std::cout << real << " + " << imag << "i" << std::endl;
    }
};

int main() {
    Complex c1(3, 4);
    Complex c2(1, 2);

    Complex sum = c1 + c2;      // calls c1.operator+(c2)
    Complex diff = c1 - c2;     // calls c1.operator-(c2)
    Complex prod = c1 * c2;     // calls c1.operator*(c2)
    Complex neg = -c1;          // calls c1.operator-()

    sum.print();   // 4 + 6i
    diff.print();  // 2 + 2i
    prod.print();  // -5 + 10i
    neg.print();   // -3 + -4i

    return 0;
}
```

## Non-Member (Friend) Implementation

For symmetric operators, non-member functions are often preferred:

```cpp
class Complex {
private:
    double real, imag;

public:
    Complex(double r = 0, double i = 0) : real(r), imag(i) {}

    // Declare friend functions
    friend Complex operator+(const Complex& lhs, const Complex& rhs);
    friend Complex operator*(double scalar, const Complex& c);
};

// Non-member operator+
Complex operator+(const Complex& lhs, const Complex& rhs) {
    return Complex(lhs.real + rhs.real, lhs.imag + rhs.imag);
}

// Scalar multiplication: 2 * complex
Complex operator*(double scalar, const Complex& c) {
    return Complex(scalar * c.real, scalar * c.imag);
}

int main() {
    Complex c(3, 4);
    Complex result = 2.0 * c;  // Works! Calls operator*(2.0, c)
    return 0;
}
```

## Why Use Non-Member for `+`?

Consider this scenario with member-only implementation:

```cpp
Complex c(1, 2);
Complex r1 = c + 5.0;    // OK: c.operator+(Complex(5.0, 0))
Complex r2 = 5.0 + c;    // ERROR: 5.0.operator+(c) - double has no operator+
```

With non-member friend:
```cpp
Complex r2 = 5.0 + c;    // OK: operator+(Complex(5.0, 0), c)
```

## Compound Assignment Operators

Implement `+=`, `-=`, etc. as members, then implement `+`, `-` using them:

```cpp
class Complex {
public:
    // Compound assignment (member)
    Complex& operator+=(const Complex& other) {
        real += other.real;
        imag += other.imag;
        return *this;
    }

    // Binary + implemented using +=
    friend Complex operator+(Complex lhs, const Complex& rhs) {
        lhs += rhs;  // reuse +=
        return lhs;
    }
};
```

> [!tip] Best Practice
> Implement compound assignment operators first, then define binary operators in terms of them. This avoids code duplication.

## Key Takeaways

- Return a new object for binary arithmetic operators (`+`, `-`, `*`, `/`)
- Return `*this` by reference for compound assignment operators (`+=`, `-=`)
- Use `const` on member functions that don't modify the object
- Consider non-member implementation for symmetric operations
- Implement binary operators using compound assignment to reduce duplication

## Common Mistakes

> [!warning] Common Mistakes
> 1. **Returning by reference** - `operator+` should return by value, not reference (the result is a new temporary object)
> 2. **Forgetting `const`** - Binary operators shouldn't modify operands
> 3. **Not handling self-assignment** - More relevant for `=` but good habit

## Related Topics

- [[04_assignment_operator|Assignment Operator]]
- [[06_increment_decrement|Increment/Decrement Operators]]
