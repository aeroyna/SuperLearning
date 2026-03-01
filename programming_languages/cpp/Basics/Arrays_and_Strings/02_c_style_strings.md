# C-Style Strings

In C, a string is simply an array of characters that is terminated by a null character (`\0`). C++ also supports these C-style strings.

## Declaration and Initialization

```cpp
char greeting[] = "Hello";
```

This creates an array of characters of size 6 (5 for "Hello" and 1 for the null terminator).

```
'H' 'e' 'l' 'l' 'o' '\0'
```

You can also use a pointer to a string literal.

```cpp
const char* greeting = "Hello";
```
This is generally safer as it prevents modification of the string literal.

## String Manipulation Functions

C++ provides a set of functions for manipulating C-style strings in the `<cstring>` header.

| Function   | Description                               |
|------------|-------------------------------------------|
| `strlen`   | Returns the length of a string            |
| `strcpy`   | Copies one string to another              |
| `strcat`   | Concatenates (appends) one string to another |
| `strcmp`   | Compares two strings                      |

### Example

```cpp
#include <iostream>
#include <cstring>

int main() {
    char str1[20] = "Hello";
    char str2[] = " World";
    char str3[20];

    // strlen
    std::cout << "Length of str1: " << strlen(str1) << std::endl;

    // strcpy
    strcpy(str3, str1);
    std::cout << "str3: " << str3 << std::endl;

    // strcat
    strcat(str1, str2);
    std::cout << "str1 after concatenation: " << str1 << std::endl;

    // strcmp
    int result = strcmp(str3, "Hello");
    if (result == 0) {
        std::cout << "str3 is equal to \"Hello\"" << std::endl;
    }

    return 0;
}
```

## Dangers of C-Style Strings

C-style strings are prone to errors, such as buffer overflows. For example, if you try to copy a string that is larger than the destination buffer, `strcpy` will write past the end of the buffer, leading to undefined behavior.

```cpp
char dest[5];
strcpy(dest, "This is a very long string"); // buffer overflow!
```

Because of these dangers, it is highly recommended to use `std::string` in C++ for string manipulation.

```