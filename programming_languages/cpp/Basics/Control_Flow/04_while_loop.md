# `while` Loop

The `while` loop executes a block of code as long as a specified condition is true.

## Syntax

```cpp
while (condition) {
    // code to be executed
}
```

*   **condition:** Checked before each iteration. If it is `true`, the loop continues; otherwise, the loop terminates.

### Example

```cpp
#include <iostream>

int main() {
    int i = 0;
    while (i < 5) {
        std::cout << "i = " << i << std::endl;
        ++i;
    }
    return 0;
}
```

## When to use `while` vs. `for`

*   Use a `for` loop when you know the number of iterations in advance.
*   Use a `while` loop when you don't know the number of iterations, and the loop depends on some other condition.

### Example: Reading user input until a specific value is entered

```cpp
#include <iostream>

int main() {
    int number;
    std::cout << "Enter a number (or -1 to quit): ";
    std::cin >> number;

    while (number != -1) {
        std::cout << "You entered: " << number << std::endl;
        std::cout << "Enter a number (or -1 to quit): ";
        std::cin >> number;
    }

    std::cout << "Goodbye!" << std::endl;

    return 0;
}
```
