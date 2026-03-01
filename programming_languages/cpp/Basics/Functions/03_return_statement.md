# `return` Statement

The `return` statement is used to end the execution of a function and return a value to the caller.

## Returning a Value

A function that is not declared as `void` must return a value. The `return` statement is used for this purpose.

### Syntax

```cpp
return expression;
```

The `expression` is converted to the function's return type.

### Example

```cpp
#include <iostream>

int add(int a, int b) {
    return a + b;
}

int main() {
    int sum = add(5, 3);
    std::cout << "The sum is: " << sum << std::endl;
    return 0;
}
```

## `void` Functions

A function that is declared as `void` does not return a value. The `return` statement can be used without an expression to exit the function early.

### Example

```cpp
#include <iostream>

void print_positive(int n) {
    if (n <= 0) {
        return; // exit the function if n is not positive
    }
    std::cout << n << " is a positive number." << std::endl;
}

int main() {
    print_positive(10);
    print_positive(-5);
    return 0;
}
```

## Returning by Reference

You can also return a value by reference. This can be useful when you want to return a large object without copying it.

### Example

```cpp
#include <iostream>

int& get_max(int& a, int& b) {
    return (a > b) ? a : b;
}

int main() {
    int x = 10;
    int y = 20;

    get_max(x, y) = 30; // modify the larger value

    std::cout << "x = " << x << std::endl;
    std::cout << "y = " << y << std::endl;

    return 0;
}
```

Output:
```
x = 10
y = 30
```

**Warning:** Do not return a reference to a local variable. The local variable is destroyed when the function exits, and the returned reference will be dangling (pointing to invalid memory).
