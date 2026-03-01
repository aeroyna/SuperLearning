# Arithmetic Operators

Arithmetic operators are used to perform common mathematical operations.

| Operator | Description      | Example     |
|----------|------------------|-------------|
| `+`      | Addition         | `a + b`     |
| `-`      | Subtraction      | `a - b`     |
| `*`      | Multiplication   | `a * b`     |
| `/`      | Division         | `a / b`     |
| `%`      | Modulo (remainder) | `a % b`     |
| `++`     | Increment        | `a++` or `++a` |
| `--`     | Decrement        | `a--` or `--a` |

### Example

```cpp
#include <iostream>

int main() {
    int a = 10;
    int b = 3;

    std::cout << "a + b = " << (a + b) << std::endl;
    std::cout << "a - b = " << (a - b) << std::endl;
    std::cout << "a * b = " << (a * b) << std::endl;
    std::cout << "a / b = " << (a / b) << std::endl; // Integer division
    std::cout << "a % b = " << (a % b) << std::endl;

    int c = 5;
    c++; // c becomes 6
    std::cout << "c++ = " << c << std::endl;

    int d = 5;
    d--; // d becomes 4
    std::cout << "d-- = " << d << std::endl;

    return 0;
}
```

## Integer Division

When you divide two integers, the result is also an integer. The fractional part is discarded.

```cpp
int x = 10 / 3; // x will be 3
```

To get a floating-point result, at least one of the operands must be a floating-point number.

```cpp
double y = 10.0 / 3; // y will be 3.333...
```

## Increment and Decrement Operators

*   **Prefix Increment (`++a`):** Increments the value of the variable and then returns the new value.
*   **Postfix Increment (`a++`):** Returns the current value of the variable and then increments it.

```cpp
int a = 5;
int b = ++a; // a becomes 6, b becomes 6

int c = 5;
int d = c++; // d becomes 5, c becomes 6
```

The same logic applies to the decrement operator (`--`).
