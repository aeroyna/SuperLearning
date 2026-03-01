# Miscellaneous Operators

There are a few other important operators in C++.

| Operator          | Description                                    | Example               |
|-------------------|------------------------------------------------|-----------------------|
| `sizeof`          | Returns the size of a variable or data type    | `sizeof(int)`         |
| `?:` (Ternary)    | Conditional operator                           | `x > y ? x : y`       |
| `,` (Comma)       | Evaluates multiple expressions               | `a = (b=3, b+2)`      |
| `.` (Dot)         | Accesses members of a struct or class          | `s.member`            |
| `->` (Arrow)      | Accesses members of a struct or class through a pointer | `ptr->member` |
| `&` (Address-of)  | Returns the memory address of a variable       | `&x`                  |
| `*` (Dereference) | Dereferences a pointer                         | `*ptr`                |

## `sizeof` Operator

We have already seen this operator in the "Data Types" section. It returns the size of a data type or a variable in bytes.

## Ternary Operator (`?:`)

The ternary operator is a shorthand for an `if-else` statement.

```cpp
condition ? expression1 : expression2;
```

If `condition` is `true`, `expression1` is evaluated; otherwise, `expression2` is evaluated.

### Example

```cpp
#include <iostream>

int main() {
    int a = 10;
    int b = 20;
    int max = (a > b) ? a : b;

    std::cout << "The maximum is: " << max << std::endl;

    return 0;
}
```

## Comma Operator (`,`)

The comma operator is used to separate two or more expressions. The expressions are evaluated from left to right, and the result of the entire expression is the value of the rightmost expression.

```cpp
int x = (2, 3); // x will be 3
```

It is often used in `for` loops to initialize and update multiple variables.

```cpp
for (int i = 0, j = 10; i < j; ++i, --j) {
    // ...
}
```

## Dot (`.`) and Arrow (`->`) Operators

These operators are used to access members of `struct`s and `class`es. We will see more of them in the "Object-Oriented Programming" section.

## Address-of (`&`) and Dereference (`*`) Operators

These operators are used with pointers. We will cover them in detail in the "Pointers and References" section.
