# Generic Lambdas (C++14)

C++14 introduced generic lambdas, which allow you to use `auto` in the parameter list of a lambda. This makes the lambda act like a function template.

## Syntax

```cpp
auto my_lambda = [](auto a, auto b) {
    return a + b;
};
```

This is roughly equivalent to:

```cpp
struct {
    template <typename T, typename U>
    auto operator()(T a, U b) const {
        return a + b;
    }
} my_lambda;
```

## Example

A generic lambda can be used to operate on different types of data.

```cpp
#include <iostream>
#include <string>

int main() {
    auto add = [](auto a, auto b) {
        return a + b;
    };

    std::cout << add(3, 4) << std::endl;         // 7
    std::cout << add(3.5, 4.5) << std::endl;     // 8.0
    std::cout << add(std::string("hello"), std::string(" world")) << std::endl; // "hello world"

    return 0;
}
```

## Variadic Generic Lambdas (C++14)

You can combine generic lambdas with variadic templates to create a lambda that can take any number of arguments of any type.

```cpp
#include <iostream>

int main() {
    auto print_all = [](auto... args) {
        ((std::cout << args << " "), ...); // C++17 fold expression
        std::cout << std::endl;
    };

    print_all(1, 2.5, "hello");

    return 0;
}
```

Generic lambdas make the lambda syntax even more powerful and concise, allowing you to write highly generic code with ease. They are a great example of how C++ continues to evolve to support more modern and expressive programming styles.
