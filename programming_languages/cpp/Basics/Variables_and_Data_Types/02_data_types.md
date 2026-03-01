# Data Types in C++

Data types define the type of data a variable can hold. C++ has a rich set of built-in data types.

## Primitive Data Types

These are the fundamental data types in C++.

| Type      | Keyword     | Size (in bytes) | Range                                                           |
|-----------|-------------|-----------------|-----------------------------------------------------------------|
| Integer   | `int`       | 4               | -2,147,483,648 to 2,147,483,647                                  |
| Character | `char`      | 1               | -128 to 127 or 0 to 255                                         |
| Boolean   | `bool`      | 1               | `true` or `false`                                               |
| Floating Point | `float`| 4               | Approximately ±3.4e-38 to ±3.4e+38 with 7 significant digits     |
| Double Floating Point | `double` | 8      | Approximately ±1.7e-308 to ±1.7e+308 with 15 significant digits |
| Void      | `void`      | -               | Represents the absence of a type                                |
| Wide Character | `wchar_t` | 2 or 4        | Used for characters in character sets that don't fit in a `char` |

*Note: The size and range of these data types can vary depending on the system and the compiler.*

### Example

```cpp
#include <iostream>

int main() {
    int age = 25;
    char grade = 'A';
    bool isStudent = true;
    float average = 85.5f; // 'f' suffix for float literals
    double pi = 3.14159;

    std::cout << "Age: " << age << std::endl;
    std::cout << "Grade: " << grade << std::endl;
    std::cout << "Is a student? " << isStudent << std::endl;
    std::cout << "Average: " << average << std::endl;
    std::cout << "PI: " << pi << std::endl;

    return 0;
}
```

## `sizeof` Operator

You can use the `sizeof` operator to get the size of a data type or a variable in bytes.

```cpp
#include <iostream>

int main() {
    std::cout << "Size of int: " << sizeof(int) << " bytes" << std::endl;
    std::cout << "Size of char: " << sizeof(char) << " bytes" << std::endl;
    std::cout << "Size of bool: " << sizeof(bool) << " bytes" << std::endl;
    std::cout << "Size of float: " << sizeof(float) << " bytes" << std::endl;
    std::cout << "Size of double: " << sizeof(double) << " bytes" << std::endl;

    return 0;
}
```
