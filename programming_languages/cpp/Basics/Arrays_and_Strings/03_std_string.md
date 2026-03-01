# `std::string`

The C++ Standard Library provides a `std::string` class that is a much safer and more convenient way to work with strings than C-style strings.

To use `std::string`, you need to include the `<string>` header.

## Declaration and Initialization

```cpp
#include <string>

std::string s1; // empty string
std::string s2 = "Hello";
std::string s3("World");
std::string s4(s2); // copy constructor
```

## Common Operations

### Concatenation

You can use the `+` operator to concatenate strings.

```cpp
std::string first_name = "John";
std::string last_name = "Doe";
std::string full_name = first_name + " " + last_name;
```

### Accessing Characters

You can access individual characters using the `[]` operator or the `at()` member function. `at()` provides bounds checking and throws an exception if the index is out of range.

```cpp
std::string s = "Hello";
char c1 = s[0]; // 'H'
char c2 = s.at(1); // 'e'
```

### Length/Size

The `length()` or `size()` member functions return the number of characters in the string.

```cpp
std::string s = "Hello";
std::cout << "Length: " << s.length() << std::endl; // 5
```

### Substrings

The `substr()` member function returns a substring.

```cpp
std::string s = "Hello, World!";
std::string sub = s.substr(7, 5); // "World" (start at index 7, length 5)
```

### Find

The `find()` member function searches for a substring.

```cpp
std::string s = "Hello, World!";
size_t pos = s.find("World");
if (pos != std::string::npos) { // npos is a special value indicating not found
    std::cout << "Found at index: " << pos << std::endl;
}
```

### Example

```cpp
#include <iostream>
#include <string>

int main() {
    std::string greeting = "Hello";
    greeting += " World"; // append

    std::cout << greeting << std::endl;
    std::cout << "Length: " << greeting.length() << std::endl;
    std::cout << "Character at index 1: " << greeting[1] << std::endl;

    std::string sub = greeting.substr(0, 5);
    std::cout << "Substring: " << sub << std::endl;

    return 0;
}
```

## `std::string` vs. C-style strings

| `std::string`                                 | C-style string                            |
|-----------------------------------------------|-------------------------------------------|
| Manages its own memory (no buffer overflows)  | Manual memory management (risk of overflows) |
| Provides a rich set of member functions       | Relies on functions from `<cstring>`      |
| Can be easily resized                         | Fixed size                                |
| Slower than C-style strings in some cases     | Can be faster                           |

In general, you should always prefer `std::string` over C-style strings in C++.
