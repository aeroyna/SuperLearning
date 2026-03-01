# Practice Problems: Control Flow

## Problem 1: Factorial

Write a C++ program to find the factorial of a number. The factorial of a non-negative integer `n`, denoted by `n!`, is the product of all positive integers less than or equal to `n`.

*   Example: `5! = 5 * 4 * 3 * 2 * 1 = 120`

### Solution

```cpp
#include <iostream>

int main() {
    int n;
    long long factorial = 1;

    std::cout << "Enter a positive integer: ";
    std::cin >> n;

    if (n < 0) {
        std::cout << "Error! Factorial of a negative number doesn't exist.";
    } else {
        for (int i = 1; i <= n; ++i) {
            factorial *= i;
        }
        std::cout << "Factorial of " << n << " = " << factorial;
    }

    return 0;
}
```

## Problem 2: Prime Number

Write a C++ program to check whether a number is prime or not. A prime number is a natural number greater than 1 that has no positive divisors other than 1 and itself.

### Solution

```cpp
#include <iostream>
#include <cmath>

int main() {
    int n;
    bool isPrime = true;

    std::cout << "Enter a positive integer: ";
    std::cin >> n;

    if (n <= 1) {
        isPrime = false;
    } else {
        for (int i = 2; i <= sqrt(n); ++i) {
            if (n % i == 0) {
                isPrime = false;
                break;
            }
        }
    }

    if (isPrime) {
        std::cout << n << " is a prime number.";
    } else {
        std::cout << n << " is not a prime number.";
    }

    return 0;
}
```

## Problem 3: Palindrome

Write a C++ program to check whether a number is a palindrome or not. A palindrome number is a number that remains the same when its digits are reversed.

*   Example: 121 is a palindrome, but 123 is not.

### Solution

```cpp
#include <iostream>

int main() {
    int n, reversedNumber = 0, remainder, originalNumber;

    std::cout << "Enter an integer: ";
    std::cin >> n;

    originalNumber = n;

    while (n != 0) {
        remainder = n % 10;
        reversedNumber = reversedNumber * 10 + remainder;
        n /= 10;
    }

    if (originalNumber == reversedNumber) {
        std::cout << originalNumber << " is a palindrome.";
    } else {
        std::cout << originalNumber << " is not a palindrome.";
    }

    return 0;
}
```
