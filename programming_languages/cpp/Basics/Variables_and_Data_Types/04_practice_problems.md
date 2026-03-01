# Practice Problems: Variables and Data Types

## Problem 1: Variable Swap

Write a C++ program that swaps the values of two integer variables.

### Solution

```cpp
#include <iostream>

int main() {
    int a = 5;
    int b = 10;
    int temp;

    std::cout << "Before swap: a = " << a << ", b = " << b << std::endl;

    temp = a;
    a = b;
    b = temp;

    std::cout << "After swap: a = " << a << ", b = " << b << std::endl;

    return 0;
}
```

## Problem 2: Area of a Circle

Write a C++ program that calculates the area of a circle. The program should ask the user to enter the radius of the circle.

*   Formula for the area of a circle: `Area = PI * r^2`

### Solution

```cpp
#include <iostream>

int main() {
    const double PI = 3.14159;
    double radius;
    double area;

    std::cout << "Enter the radius of the circle: ";
    std::cin >> radius;

    area = PI * radius * radius;

    std::cout << "The area of the circle is: " << area << std::endl;

    return 0;
}
```

## Problem 3: Character to ASCII

Write a C++ program that takes a character as input from the user and prints its ASCII value.

### Solution

```cpp
#include <iostream>

int main() {
    char c;

    std::cout << "Enter a character: ";
    std::cin >> c;

    std::cout << "The ASCII value of " << c << " is " << int(c) << std::endl;

    return 0;
}
```
