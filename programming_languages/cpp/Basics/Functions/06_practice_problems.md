# Practice Problems: Functions

## Problem 1: `max` function

Write a function that takes two integers as input and returns the maximum of the two.

### Solution

```cpp
#include <iostream>

int max(int a, int b) {
    return (a > b) ? a : b;
}

int main() {
    int x = 10;
    int y = 20;
    std::cout << "The maximum of " << x << " and " << y << " is " << max(x, y) << std::endl;
    return 0;
}
```

## Problem 2: Fibonacci Sequence (Recursive)

Write a recursive function to generate the nth Fibonacci number. The Fibonacci sequence is a series of numbers where each number is the sum of the two preceding ones, starting from 0 and 1.

*   `F(0) = 0`, `F(1) = 1`
*   `F(n) = F(n-1) + F(n-2)` for `n > 1`

### Solution

```cpp
#include <iostream>

int fibonacci(int n) {
    if (n <= 1) {
        return n;
    }
    return fibonacci(n - 1) + fibonacci(n - 2);
}

int main() {
    int n = 10;
    std::cout << "The " << n << "th Fibonacci number is " << fibonacci(n) << std::endl;
    return 0;
}
```
*Note: This recursive solution is not efficient for large values of `n` due to repeated calculations. An iterative solution would be better in that case.*

## Problem 3: Power Function

Write a function that calculates the power of a number. The function should take two arguments, `base` and `exponent`, and return `base` raised to the power of `exponent`.

### Solution

```cpp
#include <iostream>

double power(double base, int exponent) {
    double result = 1.0;
    for (int i = 0; i < exponent; ++i) {
        result *= base;
    }
    return result;
}

int main() {
    double base = 2.0;
    int exponent = 10;
    std::cout << base << " raised to the power of " << exponent << " is " << power(base, exponent) << std::endl;
    return 0;
}
```
