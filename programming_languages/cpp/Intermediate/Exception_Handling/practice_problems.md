# Practice Problems: Exception Handling

## Problem 1: Custom Exception

Create a custom exception class `MyException` that inherits from `std::exception`. Write a function that throws this exception, and catch it in `main`.

### Solution

```cpp
#include <iostream>
#include <exception>

class MyException : public std::exception {
public:
    const char* what() const noexcept override {
        return "This is my custom exception!";
    }
};

void throw_my_exception() {
    throw MyException();
}

int main() {
    try {
        throw_my_exception();
    } catch (const MyException& e) {
        std::cerr << "Caught an exception: " << e.what() << std::endl;
    }
    return 0;
}
```

## Problem 2: Exception Safety in a Class

You have a `SafeArray` class that wraps a dynamically allocated array. Ensure that the class is exception-safe. Specifically, make sure that if the `new` operator in the constructor throws a `std::bad_alloc` exception, no memory is leaked. (Hint: this is already handled by the language, but it's a good thought exercise). More importantly, implement an `at()` method that throws `std::out_of_range` for invalid indices.

### Solution

```cpp
#include <iostream>
#include <stdexcept>
#include <memory>

template<typename T>
class SafeArray {
private:
    std::unique_ptr<T[]> data;
    size_t size;

public:
    explicit SafeArray(size_t s) : size(s) {
        // Using unique_ptr makes this automatically exception-safe.
        // If `new T[s]` fails and throws bad_alloc, the unique_ptr is
        // not constructed, and no memory is leaked.
        data = std::make_unique<T[]>(s);
    }

    T& at(size_t index) {
        if (index >= size) {
            throw std::out_of_range("Index out of range");
        }
        return data[index];
    }

    const T& at(size_t index) const {
        if (index >= size) {
            throw std::out_of_range("Index out of range");
        }
        return data[index];
    }

    size_t getSize() const {
        return size;
    }
};

int main() {
    try {
        SafeArray<int> arr(10);
        arr.at(5) = 100;
        std::cout << "arr.at(5) = " << arr.at(5) << std::endl;

        // This should throw an exception
        arr.at(10) = 200;

    } catch (const std::out_of_range& e) {
        std::cerr << "Error: " << e.what() << std::endl;
    } catch (const std::bad_alloc& e) {
        std::cerr << "Error: Failed to allocate memory: " << e.what() << std::endl;
    }

    return 0;
}
```
This solution uses `std::unique_ptr` to demonstrate how RAII makes exception safety much easier to achieve. The `unique_ptr` will automatically handle the deallocation of the memory, even if an exception is thrown later.
