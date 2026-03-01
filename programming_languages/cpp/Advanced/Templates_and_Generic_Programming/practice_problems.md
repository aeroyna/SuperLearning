# Practice Problems: Templates and Generic Programming

## Problem 1: Generic `min` function

Write a function template `min_val` that takes two arguments and returns the smaller of the two. Test it with `int`, `double`, and `std::string`.

### Solution

```cpp
#include <iostream>
#include <string>

template <typename T>
T min_val(T a, T b) {
    return (a < b) ? a : b;
}

int main() {
    std::cout << "Min of 3 and 7 is " << min_val(3, 7) << std::endl;
    std::cout << "Min of 3.14 and 2.71 is " << min_val(3.14, 2.71) << std::endl;
    std::cout << "Min of \"apple\" and \"orange\" is " << min_val(std::string("apple"), std::string("orange")) << std::endl;
    return 0;
}
```

## Problem 2: Stack Class Template

Create a class template `Stack` that implements a simple stack data structure. It should have the following methods:

*   `push(T element)`: Pushes an element onto the stack.
*   `pop()`: Removes the top element from the stack.
*   `top() const`: Returns a reference to the top element.
*   `is_empty() const`: Returns `true` if the stack is empty.

Use a `std::vector` as the underlying container.

### Solution

```cpp
#include <iostream>
#include <vector>
#include <stdexcept>

template <typename T>
class Stack {
private:
    std::vector<T> elements;

public:
    void push(T element) {
        elements.push_back(element);
    }

    void pop() {
        if (is_empty()) {
            throw std::out_of_range("Stack is empty");
        }
        elements.pop_back();
    }

    T& top() {
        if (is_empty()) {
            throw std::out_of_range("Stack is empty");
        }
        return elements.back();
    }

    const T& top() const {
        if (is_empty()) {
            throw std::out_of_range("Stack is empty");
        }
        return elements.back();
    }

    bool is_empty() const {
        return elements.empty();
    }
};

int main() {
    Stack<int> int_stack;
    int_stack.push(10);
    int_stack.push(20);
    std::cout << "Top element: " << int_stack.top() << std::endl;
    int_stack.pop();
    std::cout << "Top element: " << int_stack.top() << std::endl;

    Stack<std::string> string_stack;
    string_stack.push("hello");
    string_stack.push("world");
    std::cout << "Top element: " << string_stack.top() << std::endl;

    return 0;
}
```

## Problem 3: Variadic `sum` function

Write a variadic template function `sum` that calculates the sum of all its arguments. You can use either recursion or a C++17 fold expression.

### Solution (using fold expression)

```cpp
#include <iostream>

template <typename... Args>
auto sum(Args... args) {
    return (args + ...);
}

int main() {
    std::cout << "Sum of 1, 2, 3: " << sum(1, 2, 3) << std::endl;
    std::cout << "Sum of 1.5, 2.5, 3.5: " << sum(1.5, 2.5, 3.5) << std::endl;
    std::cout << "Sum of 1, 2.5, 3: " << sum(1, 2.5, 3) << std::endl;
    return 0;
}
```