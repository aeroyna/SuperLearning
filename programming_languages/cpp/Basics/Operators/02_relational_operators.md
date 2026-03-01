# Relational Operators

Relational operators are used to compare two values. The result of a relational operation is a boolean value (`true` or `false`).

| Operator | Description              | Example     |
|----------|--------------------------|-------------|
| `==`     | Equal to                 | `a == b`    |
| `!=`     | Not equal to             | `a != b`    |
| `>`      | Greater than             | `a > b`     |
| `<`      | Less than                | `a < b`     |
| `>=`     | Greater than or equal to | `a >= b`    |
| `<=`     | Less than or equal to    | `a <= b`    |

### Example

```cpp
#include <iostream>

int main() {
    int a = 10;
    int b = 5;

    std::cout << std::boolalpha; // Print booleans as "true" or "false"

    std::cout << "a == b: " << (a == b) << std::endl;
    std::cout << "a != b: " << (a != b) << std::endl;
    std::cout << "a > b: " << (a > b) << std::endl;
    std::cout << "a < b: " << (a < b) << std::endl;
    std::cout << "a >= b: " << (a >= b) << std::endl;
    std::cout << "a <= b: " << (a <= b) << std::endl;

    return 0;
}
```

## Using Relational Operators with `if` Statements

Relational operators are commonly used in `if` statements to control the flow of the program.

```cpp
#include <iostream>

int main() {
    int age;
    std::cout << "Enter your age: ";
    std::cin >> age;

    if (age >= 18) {
        std::cout << "You are eligible to vote." << std::endl;
    } else {
        std::cout << "You are not eligible to vote." << std::endl;
    }

    return 0;
}
```
