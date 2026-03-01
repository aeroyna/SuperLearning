# Dynamic Memory Allocation

So far, we have been working with static memory allocation, where the size of variables is fixed at compile time. Dynamic memory allocation allows you to allocate memory at runtime.

The memory for dynamically allocated variables is allocated from the **heap**, whereas the memory for static variables is allocated from the **stack**.

## `new` and `delete` Operators

C++ provides the `new` and `delete` operators for dynamic memory allocation.

### `new` Operator

The `new` operator allocates memory from the heap and returns a pointer to the allocated memory.

```cpp
type* pointer = new type;
```

### `delete` Operator

The `delete` operator deallocates the memory that was previously allocated by `new`.

```cpp
delete pointer;
```

**Important:** For every `new`, there must be a corresponding `delete`. If you fail to deallocate memory, it will result in a **memory leak**.

### Example

```cpp
#include <iostream>

int main() {
    int* ptr = new int; // allocate memory for an integer
    *ptr = 20;

    std::cout << "Value: " << *ptr << std::endl;

    delete ptr; // deallocate the memory
    ptr = nullptr; // good practice to set the pointer to nullptr after deleting

    return 0;
}
```

## Allocating Arrays

You can also allocate arrays dynamically.

```cpp
type* pointer = new type[size];
```

To deallocate an array, you must use the `delete[]` operator.

```cpp
delete[] pointer;
```

### Example

```cpp
#include <iostream>

int main() {
    int size;
    std::cout << "Enter the size of the array: ";
    std::cin >> size;

    int* arr = new int[size];

    for (int i = 0; i < size; ++i) {
        arr[i] = i * 10;
    }

    for (int i = 0; i < size; ++i) {
        std::cout << arr[i] << " ";
    }
    std::cout << std::endl;

    delete[] arr;
    arr = nullptr;

    return 0;
}
```

## Dangers of Manual Memory Management

Manual memory management with `new` and `delete` is error-prone:

*   **Memory Leaks:** Forgetting to `delete` memory.
*   **Dangling Pointers:** Using a pointer after the memory it points to has been deallocated.
*   **Double Deletes:** Trying to `delete` the same memory twice.

To avoid these problems, modern C++ encourages the use of **smart pointers** (`std::unique_ptr`, `std::shared_ptr`), which automatically manage memory. We will cover smart pointers in the "Memory Management" section of the intermediate topics.
