# Logical Operators

Logical operators are used to combine two or more conditions.

| Operator | Description | Example               |
|----------|-------------|-----------------------|
| `&&`     | Logical AND | `(a > b) && (a > c)`  |
| `||`     | Logical OR  | `(a > b) || (a > c)`  |
| `!`      | Logical NOT | `!(a == b)`           |

## Truth Tables

### Logical AND (`&&`)

| A     | B     | A && B |
|-------|-------|--------|
| true  | true  | true   |
| true  | false | false  |
| false | true  | false  |
| false | false | false  |

### Logical OR (`||`)

| A     | B     | A \|\| B |
|-------|-------|----------|
| true  | true  | true     |
| true  | false | true     |
| false | true  | true     |
| false | false | false    |

### Logical NOT (`!`)

| A     | !A    |
|-------|-------|
| true  | false |
| false | true  |

### Example

```cpp
#include <iostream>

int main() {
    int a = 10;
    int b = 5;
    int c = 20;

    std::cout << std::boolalpha;

    // Logical AND
    std::cout << "(a > b) && (a < c): " << ((a > b) && (a < c)) << std::endl;

    // Logical OR
    std::cout << "(a > b) || (a > c): " << ((a > b) || (a > c)) << std::endl;

    // Logical NOT
    std::cout << "!(a == b): " << !(a == b) << std::endl;

    return 0;
}
```

## Short-Circuit Evaluation

Logical operators in C++ use short-circuit evaluation. This means that the second operand is evaluated only if the first one is not enough to determine the result.

*   In `A && B`, if `A` is `false`, `B` is not evaluated because the whole expression is `false`.
*   In `A || B`, if `A` is `true`, `B` is not evaluated because the whole expression is `true`.

This is important to remember when the second operand has side effects (e.g., a function call or an increment operator).
