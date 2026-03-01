# Practice Problems: Move Semantics and Rvalue References

## Problem 1: Add Move Semantics to a Class

You have a `Buffer` class that manages a dynamically allocated block of memory. Add a move constructor and a move assignment operator to this class to make it more efficient.

```cpp
#include <iostream>
#include <algorithm> // for std::swap

class Buffer {
private:
    char* data;
    size_t size;

public:
    // Constructor
    Buffer(size_t s) : size(s) {
        data = new char[size];
        std::cout << "Buffer constructed\n";
    }

    // Copy Constructor
    Buffer(const Buffer& other) : size(other.size) {
        data = new char[size];
        std::copy(other.data, other.data + size, data);
        std::cout << "Buffer copy-constructed\n";
    }

    // Copy Assignment Operator
    Buffer& operator=(const Buffer& other) {
        if (this != &other) {
            delete[] data;
            size = other.size;
            data = new char[size];
            std::copy(other.data, other.data + size, data);
        }
        std::cout << "Buffer copy-assigned\n";
        return *this;
    }

    // Destructor
    ~Buffer() {
        delete[] data;
        std::cout << "Buffer destructed\n";
    }

    // (Move constructor and move assignment operator to be added)
};
```

### Solution

```cpp
// (Add these to the Buffer class)

// Move Constructor
Buffer(Buffer&& other) noexcept
    : data(other.data), size(other.size) {
    other.data = nullptr;
    other.size = 0;
    std::cout << "Buffer move-constructed\n";
}

// Move Assignment Operator
Buffer& operator=(Buffer&& other) noexcept {
    if (this != &other) {
        delete[] data;
        data = other.data;
        size = other.size;
        other.data = nullptr;
        other.size = 0;
    }
    std::cout << "Buffer move-assigned\n";
    return *this;
}

// Main function to test
int main() {
    std::cout << "Creating b1\n";
    Buffer b1(1024);

    std::cout << "\nCreating b2 from b1 (copy)\n";
    Buffer b2 = b1;

    std::cout << "\nCreating b3 from temporary (move)\n";
    Buffer b3 = Buffer(2048);

    std::cout << "\nMove-assigning b3 to b1\n";
    b1 = std::move(b3);
    
    std::cout << "\nEnd of main\n";
    return 0;
}
```

## Problem 2: A simple `push_back` that uses `std::move`

Write a function `append_to_vector` that takes a `std::vector<MyString>` and a `MyString` object. The function should add the `MyString` object to the vector. Create two versions of this function: one that takes an lvalue reference and copies the string, and one that takes an rvalue reference and moves the string. (Assume `MyString` is a class with move semantics, like the one from the previous problem).

### Solution

```cpp
#include <iostream>
#include <vector>
#include <string> // using std::string for simplicity, it has move semantics

// Version for lvalues (copies the string)
void append_to_vector(std::vector<std::string>& vec, const std::string& str) {
    std::cout << "Appending by copy\n";
    vec.push_back(str);
}

// Version for rvalues (moves the string)
void append_to_vector(std::vector<std::string>& vec, std::string&& str) {
    std::cout << "Appending by move\n";
    vec.push_back(std::move(str));
}

int main() {
    std::vector<std::string> my_vec;

    std::string s1 = "hello";
    append_to_vector(my_vec, s1); // calls copy version

    append_to_vector(my_vec, "world"); // calls move version

    // You can also explicitly move an lvalue
    std::string s2 = "goodbye";
    append_to_vector(my_vec, std::move(s2)); // calls move version

    for (const auto& s : my_vec) {
        std::cout << s << " ";
    }
    std::cout << std::endl;
    
    // s2 is now in a moved-from state
    std::cout << "s2 after move: \"" << s2 << "\"" << std::endl;

    return 0;
}
```
*Note: In a real-world scenario, you would just use `std::vector::push_back`, which already has overloads for lvalues and rvalues.*
