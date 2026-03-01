# Lambda Captures

The capture clause (`[]`) is what makes lambdas particularly powerful. It allows the lambda to access variables from the enclosing scope.

## Capture by Value

To capture a variable by value, you list it in the capture clause. A copy of the variable is made at the time the lambda is defined.

```cpp
#include <iostream>

int main() {
    int x = 10;
    auto my_lambda = [x]() {
        std::cout << "x inside lambda: " << x << std::endl;
    };

    x = 20;
    my_lambda(); // still prints 10, because x was captured by value

    return 0;
}
```

## Capture by Reference

To capture a variable by reference, you use the `&` symbol. This allows the lambda to modify the original variable.

```cpp
#include <iostream>

int main() {
    int x = 10;
    auto my_lambda = [&x]() {
        x = 30; // modifies the original x
    };

    my_lambda();
    std::cout << "x outside lambda: " << x << std::endl; // prints 30

    return 0;
}
```

## Default Captures

You can also specify a default capture mode:

*   `[=]`: Capture all variables from the enclosing scope by value.
*   `[&]`: Capture all variables from the enclosing scope by reference.

```cpp
int x = 10;
int y = 20;

auto lambda1 = [=]() { /* can use x and y by value */ };
auto lambda2 = [&]() { /* can use x and y by reference */ };
```

You can also mix default captures with explicit captures.

```cpp
// Capture y by value, and everything else by reference
auto lambda3 = [&, y]() { ... };

// Capture x by reference, and everything else by value
auto lambda4 = [=, &x]() { ... };
```

## `mutable` keyword

If you capture a variable by value, you cannot modify the copy inside the lambda by default. If you need to modify the copy, you can use the `mutable` keyword.

```cpp
int x = 10;
auto my_lambda = [x]() mutable {
    x = 20; // OK
    std::cout << "x inside lambda: " << x << std::endl;
};
my_lambda();
std::cout << "x outside lambda: " << x << std::endl; // still 10
```

## Generalized Lambda Capture (C++14)

C++14 introduced generalized lambda capture, which allows you to create new variables in the capture clause. This is useful for moving a variable into the lambda's state.

```cpp
#include <iostream>
#include <memory>
#include <utility>

int main() {
    auto ptr = std::make_unique<int>(10);

    // Move ptr into the lambda's state
    auto my_lambda = [p = std::move(ptr)]() {
        std::cout << "Value: " << *p << std::endl;
    };

    my_lambda();
    // std::cout << *ptr << std::endl; // Error: ptr has been moved from

    return 0;
}
```
This is a powerful feature for capturing move-only types like `std::unique_ptr`.
