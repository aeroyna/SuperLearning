# Practice Problems: Pointers and References

## Problem 1: Swap two numbers using pointers

Write a function that swaps two integer values using pointers.

### Solution

```cpp
#include <iostream>

void swap(int* a, int* b) {
    int temp = *a;
    *a = *b;
    *b = temp;
}

int main() {
    int x = 10;
    int y = 20;

    std::cout << "Before swap: x = " << x << ", y = " << y << std::endl;
    swap(&x, &y);
    std::cout << "After swap: x = " << x << ", y = " << y << std::endl;

    return 0;
}
```

## Problem 2: Find the length of a C-style string using a pointer

Write a function that takes a C-style string (i.e., a `const char*`) as input and returns its length. Do not use the `strlen` function.

### Solution

```cpp
#include <iostream>

int string_length(const char* s) {
    int length = 0;
    while (*s != '\0') {
        length++;
        s++;
    }
    return length;
}

int main() {
    const char* my_string = "Hello, World!";
    std::cout << "The length of the string is: " << string_length(my_string) << std::endl;
    return 0;
}
```

## Problem 3: Dynamically allocate an array

Write a program that asks the user for the size of an array, dynamically allocates the array, fills it with some values, prints the array, and then deallocates it.

### Solution

```cpp
#include <iostream>

int main() {
    int size;
    std::cout << "Enter the size of the array: ";
    std::cin >> size;

    // Dynamically allocate the array
    int* arr = new int[size];

    // Fill the array with values
    for (int i = 0; i < size; ++i) {
        arr[i] = i * i; // for example, the square of the index
    }

    // Print the array
    std::cout << "The array contains: ";
    for (int i = 0; i < size; ++i) {
        std::cout << arr[i] << " ";
    }
    std::cout << std::endl;

    // Deallocate the array
    delete[] arr;
    arr = nullptr;

    return 0;
}
```
