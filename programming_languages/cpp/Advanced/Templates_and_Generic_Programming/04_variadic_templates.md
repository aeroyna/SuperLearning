# Variadic Templates (C++11)

Variadic templates are templates that can take a variable number of arguments. This is a powerful feature that was introduced in C++11 and is used extensively in modern C++ libraries.

## Syntax

Variadic templates are declared using an ellipsis (`...`).

```cpp
template <typename... Args> // Args is a "template parameter pack"
void myFunction(Args... args); // args is a "function parameter pack"
```

## Unpacking the Parameter Pack

To use the arguments in a parameter pack, you need to "unpack" them. This is usually done with recursion.

### Example: A `print` function

Let's create a `print` function that can take any number of arguments and print them to the console.

```cpp
#include <iostream>

// Base case: handle the last argument
void print() {
    std::cout << std::endl;
}

// Recursive step
template <typename T, typename... Args>
void print(T first, Args... args) {
    std::cout << first << " ";
    print(args...); // recursive call with the rest of the arguments
}

int main() {
    print(1, 2.5, "hello", 'c');
    print("one argument");
    print();
    return 0;
}
```

How `print(1, 2.5, "hello")` works:

1.  `print(1, 2.5, "hello")` is called. `first` is `1`, `args` is `(2.5, "hello")`. It prints `1` and calls `print(2.5, "hello")`.
2.  `print(2.5, "hello")` is called. `first` is `2.5`, `args` is `("hello")`. It prints `2.5` and calls `print("hello")`.
3.  `print("hello")` is called. `first` is `"hello"`, `args` is `()`. It prints `"hello"` and calls `print()`.
4.  `print()` is called. This is the base case, and it prints a newline.

## Fold Expressions (C++17)

C++17 introduced fold expressions, which provide a more concise way to unpack parameter packs for certain operations.

### Example with Fold Expression

```cpp
#include <iostream>

template <typename... Args>
void print_fold(Args... args) {
    ((std::cout << args << " "), ...);
    std::cout << std::endl;
}

template <typename... Args>
auto sum(Args... args) {
    return (args + ...); // unary right fold
}

int main() {
    print_fold(1, 2.5, "hello", 'c');
    std::cout << "Sum: " << sum(1, 2, 3, 4, 5) << std::endl;
    return 0;
}
```
Fold expressions are much simpler than recursion for many common use cases.

## Other use cases for Variadic Templates

*   **`std::tuple`:** A tuple is a fixed-size collection of heterogeneous values. It is implemented using variadic templates.
*   **`std::function` and `std::bind`:** Used for type-safe function wrapping.
*   **Emplacement constructors:** `std::vector::emplace_back` and `std::make_unique`/`std::make_shared` use variadic templates and perfect forwarding to construct objects in-place, which is more efficient than copying or moving them.
