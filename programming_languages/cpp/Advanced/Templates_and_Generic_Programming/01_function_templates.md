# Function Templates

Generic programming is a style of programming in which algorithms are written in terms of types to-be-specified-later that are then instantiated when needed for specific types provided as parameters. Templates are the foundation of generic programming in C++.

## What is a Function Template?

A function template is a "template" for creating a function. It allows you to write a single function that can work with different data types.

Consider a `max` function. Without templates, you would need to write overloaded functions for each data type you want to support:

```cpp
int max(int a, int b) {
    return (a > b) ? a : b;
}

double max(double a, double b) {
    return (a > b) ? a : b;
}
```

With a function template, you can write this just once.

## Syntax

```cpp
template <typename T>
T max(T a, T b) {
    return (a > b) ? a : b;
}
```

*   `template <typename T>`: This is the template prefix. It tells the compiler that we are defining a template.
*   `T`: This is a template parameter, which represents a type. You can use any name for the template parameter, but `T` is a common convention.
*   `typename`: This keyword is used to declare a template parameter. You can also use `class` instead of `typename`.

## How it works: Instantiation

When you call a function template, the compiler **instantiates** the template for the specific data types you are using. This means that the compiler generates a regular function for those types.

### Example

```cpp
#include <iostream>
#include <string>

template <typename T>
T max_val(T a, T b) {
    return (a > b) ? a : b;
}

int main() {
    // The compiler instantiates max_val for int
    std::cout << "Max of 3 and 7 is " << max_val(3, 7) << std::endl;

    // The compiler instantiates max_val for double
    std::cout << "Max of 3.14 and 2.71 is " << max_val(3.14, 2.71) << std::endl;

    // The compiler instantiates max_val for std::string
    std::cout << "Max of \"apple\" and \"orange\" is " << max_val(std::string("apple"), std::string("orange")) << std::endl;

    return 0;
}
```

## Template Argument Deduction

In most cases, the compiler can automatically deduce the type of the template argument from the function arguments. This is called template argument deduction.

```cpp
max_val(3, 7); // T is deduced as int
```

Sometimes, you may need to explicitly specify the template argument.

```cpp
max_val<double>(3, 7.5); // T is specified as double
```

## Multiple Template Parameters

You can have multiple template parameters.

```cpp
template <typename T, typename U>
void print_pair(T a, U b) {
    std::cout << "First: " << a << ", Second: " << b << std::endl;
}

int main() {
    print_pair(10, "hello");
    print_pair(3.14, 'A');
    return 0;
}
```
Function templates are a powerful tool for writing reusable and type-safe code.

