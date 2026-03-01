# Type Modifiers in C++

Type modifiers are used to modify the meaning of the basic data types.

## List of Type Modifiers

*   `signed`: The number can be positive or negative. This is the default for integral types.
*   `unsigned`: The number can only be positive. This allows for a larger range of positive values.
*   `short`: Reduces the size of the integer type.
*   `long`: Increases the size of the integer or floating-point type.

Here's how they can be combined with `int`:

| Type                 | Size (in bytes) | Range                                     |
|----------------------|-----------------|-------------------------------------------|
| `short int`          | 2               | -32,768 to 32,767                         |
| `unsigned short int` | 2               | 0 to 65,535                               |
| `long int`           | 4 or 8          | -2,147,483,648 to 2,147,483,647 (4 bytes) |
| `unsigned long int`  | 4 or 8          | 0 to 4,294,967,295 (4 bytes)            |
| `long long int`      | 8               | -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807 |
| `unsigned long long int` | 8           | 0 to 18,446,744,073,709,551,615          |

You can also use `long` with `double`:

| Type          | Size (in bytes) | Range                                                              |
|---------------|-----------------|--------------------------------------------------------------------|
| `long double` | 10, 12, or 16   | Larger than `double`, with more precision (e.g., 19 significant digits) |

*Note: The size and range can vary depending on the system and compiler.*

### Example

```cpp
#include <iostream>

int main() {
    short int short_var = 10;
    unsigned int unsigned_var = 40000;
    long long int long_long_var = 9000000000000000000LL; // 'LL' for long long

    std::cout << "short_var: " << short_var << std::endl;
    std::cout << "unsigned_var: " << unsigned_var << std::endl;
    std::cout << "long_long_var: " << long_long_var << std::endl;

    return 0;
}
```

## `auto` Keyword (C++11 and later)

The `auto` keyword allows the compiler to automatically deduce the type of a variable from its initializer.

```cpp
auto i = 42;       // i is an int
auto d = 3.14;     // d is a double
auto s = "hello";  // s is a const char*
```

This can make the code more readable and less verbose, especially with complex types like iterators in the STL.
