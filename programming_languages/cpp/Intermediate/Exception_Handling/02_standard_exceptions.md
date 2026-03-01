# Standard Exceptions

The C++ Standard Library provides a set of standard exception classes that you can use. These are all derived from the base class `std::exception` and are defined in the `<stdexception>` header.

It is good practice to throw and catch these standard exceptions (or classes derived from them) instead of primitive types like `int` or `const char*`.

## The `std::exception` class

The `std::exception` class has a virtual member function `what()` that returns a C-style string describing the exception.

```cpp
#include <exception>

class my_exception : public std::exception {
public:
    const char* what() const noexcept override {
        return "My custom exception";
    }
};
```

When catching exceptions, you should catch them by reference to avoid object slicing.

```cpp
try {
    // ...
} catch (const std::exception& e) {
    std::cerr << "Caught exception: " << e.what() << std::endl;
}
```

## Common Standard Exception Classes

Here are some of the most common standard exception classes:

| Exception                 | Description                                           |
|---------------------------|-------------------------------------------------------|
| `std::bad_alloc`          | Thrown by `new` on failure to allocate memory.        |
| `std::bad_cast`           | Thrown by `dynamic_cast` when it fails.               |
| `std::out_of_range`       | Thrown by `at()` on containers like `std::vector`.    |
| `std::invalid_argument`   | Thrown when an invalid argument is passed to a function. |
| `std::runtime_error`      | A generic runtime error.                              |
| `std::logic_error`        | A generic logic error (a bug in the program).         |

### Example

```cpp
#include <iostream>
#include <vector>
#include <stdexcept>

void process_vector(const std::vector<int>& vec) {
    if (vec.empty()) {
        throw std::invalid_argument("Vector cannot be empty");
    }
    // ...
}

int main() {
    std::vector<int> my_vec;

    try {
        process_vector(my_vec);
        int x = my_vec.at(0); // this would also throw std::out_of_range
    } catch (const std::invalid_argument& e) {
        std::cerr << "Invalid argument error: " << e.what() << std::endl;
    } catch (const std::out_of_range& e) {
        std::cerr << "Out of range error: " << e.what() << std::endl;
    } catch (const std::exception& e) {
        std::cerr << "Some other standard exception: " << e.what() << std::endl;
    }

    return 0;
}
```

By using the standard exception classes, you make your code more consistent with the C++ standard library and other third-party libraries.
