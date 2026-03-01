# Function Overloading

Function overloading is a feature in C++ that allows you to have multiple functions with the same name but different parameters.

## How it works

The compiler determines which function to call based on the number and types of the arguments passed to it.

### Example

```cpp
#include <iostream>

void print(int i) {
    std::cout << "Printing int: " << i << std::endl;
}

void print(double d) {
    std::cout << "Printing double: " << d << std::endl;
}

void print(const char* s) {
    std::cout << "Printing string: " << s << std::endl;
}

int main() {
    print(10);
    print(3.14);
    print("Hello");
    return 0;
}
```

Output:
```
Printing int: 10
Printing double: 3.14
Printing string: Hello
```

## Rules for Function Overloading

*   The functions must have the same name.
*   The functions must have different parameters. This can be a different number of parameters, different types of parameters, or both.
*   The return type of the functions can be the same or different. However, two functions cannot be overloaded if they only differ in their return type.

### Example of what is NOT allowed

```cpp
// This is not allowed
int my_function() { ... }
void my_function() { ... }
```

## When to use Function Overloading

Function overloading can make your code more readable and intuitive. It is often used for functions that perform the same general task but on different types of data.

For example, the `+` operator is overloaded to work with `int`, `double`, `std::string`, and other types.
