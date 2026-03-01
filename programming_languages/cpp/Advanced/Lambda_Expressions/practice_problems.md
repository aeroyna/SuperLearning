# Practice Problems: Lambda Expressions

## Problem 1: Count elements that satisfy a condition

Write a function that takes a `std::vector<int>` and a `threshold` value. The function should use `std::count_if` and a lambda to count how many elements in the vector are greater than the threshold.

### Solution

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int count_greater_than(const std::vector<int>& vec, int threshold) {
    return std::count_if(vec.begin(), vec.end(), [threshold](int x) {
        return x > threshold;
    });
}

int main() {
    std::vector<int> numbers = {1, 5, 2, 8, 4, 9, 3, 7};
    int threshold = 4;
    int count = count_greater_than(numbers, threshold);
    std::cout << "There are " << count << " numbers greater than " << threshold << std::endl;
    return 0;
}
```

## Problem 2: Transform and print a vector

You have a vector of integers. Use `std::for_each` and a lambda to print the square of each element in the vector.

### Solution

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> numbers = {1, 2, 3, 4, 5};

    std::cout << "The squares are: ";
    std::for_each(numbers.begin(), numbers.end(), [](int x) {
        std::cout << x * x << " ";
    });
    std::cout << std::endl;

    return 0;
}
```

## Problem 3: Create a function that returns a lambda

Write a function `make_adder` that takes an integer `n`. The function should return a lambda that takes an integer `x` and returns `n + x`.

### Solution

```cpp
#include <iostream>
#include <functional>

// The function returns a lambda (or more precisely, a function object)
std::function<int(int)> make_adder(int n) {
    return [n](int x) {
        return n + x;
    };
}

int main() {
    auto add_5 = make_adder(5);
    auto add_10 = make_adder(10);

    std::cout << "add_5(10) = " << add_5(10) << std::endl; // 15
    std::cout << "add_10(10) = " << add_10(10) << std::endl; // 20

    return 0;
}
```
This demonstrates how lambdas can be used to create closures, which are functions that "remember" the environment in which they were created.
