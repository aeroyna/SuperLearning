# Pointers and Arrays

In C++, pointers and arrays are closely related. The name of an array is actually a pointer to the first element of the array.

### Example

```cpp
#include <iostream>

int main() {
    int numbers[] = {10, 20, 30, 40, 50};

    std::cout << "The address of the first element is: " << numbers << std::endl;
    std::cout << "The address of the first element is: " << &numbers[0] << std::endl;

    std::cout << "The value of the first element is: " << *numbers << std::endl;

    return 0;
}
```

## Pointer Arithmetic

You can use arithmetic operators with pointers. When you increment a pointer, it points to the next element of the same type.

If `ptr` is a pointer to `int`, `ptr++` will increment the address stored in `ptr` by `sizeof(int)` bytes.

### Example

```cpp
#include <iostream>

int main() {
    int numbers[] = {10, 20, 30, 40, 50};
    int* ptr = numbers;

    for (int i = 0; i < 5; ++i) {
        std::cout << "Value at address " << ptr << " is " << *ptr << std::endl;
        ptr++;
    }

    return 0;
}
```

## Accessing Array Elements with Pointers

You can use pointers to access the elements of an array in two ways:

1.  **Dereferencing:** `*(ptr + i)`
2.  **Subscripting:** `ptr[i]`

### Example

```cpp
#include <iostream>

int main() {
    int numbers[] = {10, 20, 30, 40, 50};
    int* ptr = numbers;

    std::cout << "Third element (dereferencing): " << *(ptr + 2) << std::endl;
    std::cout << "Third element (subscripting): " << ptr[2] << std::endl;

    return 0;
}
```

Both `*(numbers + 2)` and `numbers[2]` are equivalent and will give you the third element of the array.

## Pointers and C-Style Strings

Since C-style strings are arrays of characters, you can use pointers to work with them.

```cpp
#include <iostream>

int main() {
    const char* message = "Hello, World!";
    
    while (*message != '\0') {
        std::cout << *message;
        message++;
    }
    std::cout << std::endl;

    return 0;
}
```
This program iterates through the string using a pointer and prints each character until it reaches the null terminator.
