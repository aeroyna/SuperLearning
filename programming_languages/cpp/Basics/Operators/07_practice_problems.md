# Practice Problems: Operators

## Problem 1: Even or Odd

Write a C++ program that takes an integer as input and prints whether it is even or odd.

### Solution

```cpp
#include <iostream>

int main() {
    int num;
    std::cout << "Enter an integer: ";
    std::cin >> num;

    if (num % 2 == 0) {
        std::cout << num << " is even." << std::endl;
    } else {
        std::cout << num << " is odd." << std::endl;
    }

    return 0;
}
```

## Problem 2: Leap Year

Write a C++ program to check whether a year is a leap year or not.

A year is a leap year if it is divisible by 4, except for end-of-century years, which must be divisible by 400. This means that the year 2000 was a leap year, but 1900 was not.

### Solution

```cpp
#include <iostream>

int main() {
    int year;
    std::cout << "Enter a year: ";
    std::cin >> year;

    if ((year % 4 == 0 && year % 100 != 0) || (year % 400 == 0)) {
        std::cout << year << " is a leap year." << std::endl;
    } else {
        std::cout << year << " is not a leap year." << std::endl;
    }

    return 0;
}
```

## Problem 3: Swapping without a temporary variable

Write a C++ program to swap two numbers without using a third variable.

### Solution (using arithmetic operators)

```cpp
#include <iostream>

int main() {
    int a = 10;
    int b = 20;

    std::cout << "Before swap: a = " << a << ", b = " << b << std::endl;

    a = a + b;
    b = a - b;
    a = a - b;

    std::cout << "After swap: a = " << a << ", b = " << b << std::endl;

    return 0;
}
```

### Solution (using bitwise XOR operator)

```cpp
#include <iostream>

int main() {
    int a = 10;
    int b = 20;

    std::cout << "Before swap: a = " << a << ", b = " << b << std::endl;

    a = a ^ b;
    b = a ^ b;
    a = a ^ b;

    std::cout << "After swap: a = " << a << ", b = " << b << std::endl;

    return 0;
}
```
