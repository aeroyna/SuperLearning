# `do-while` Loop

The `do-while` loop is similar to the `while` loop, but the condition is checked at the end of the loop. This means that the code block is executed at least once.

## Syntax

```cpp
do {
    // code to be executed
} while (condition);
```

*   **condition:** Checked at the end of each iteration. If it is `true`, the loop continues; otherwise, the loop terminates.

### Example

```cpp
#include <iostream>

int main() {
    int i = 0;
    do {
        std::cout << "i = " << i << std::endl;
        ++i;
    } while (i < 5);

    return 0;
}
```

## `do-while` vs. `while`

The main difference is that the `do-while` loop is guaranteed to execute at least once, even if the condition is false from the beginning.

### Example: Menu-driven program

A `do-while` loop is often used for menu-driven programs where you want to display the menu at least once.

```cpp
#include <iostream>

int main() {
    int choice;
    do {
        std::cout << "1. Play Game" << std::endl;
        std::cout << "2. Load Game" << std::endl;
        std::cout << "3. Quit" << std::endl;
        std::cout << "Enter your choice: ";
        std::cin >> choice;

        switch (choice) {
            case 1:
                std::cout << "Starting a new game..." << std::endl;
                break;
            case 2:
                std::cout << "Loading game..." << std::endl;
                break;
            case 3:
                std::cout << "Quitting..." << std::endl;
                break;
            default:
                std::cout << "Invalid choice. Please try again." << std::endl;
        }
    } while (choice != 3);

    return 0;
}
```
