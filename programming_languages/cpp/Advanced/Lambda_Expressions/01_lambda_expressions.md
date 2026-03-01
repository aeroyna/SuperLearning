# Lambda Expressions (C++11)

A lambda expression is a convenient way to define an anonymous function object (a "functor") right at the location where it is needed. Lambdas are especially useful when working with STL algorithms.

## Syntax

The basic syntax of a lambda expression is:

```cpp
[captures](parameters) -> return_type {
    // function body
}
```

*   **`[captures]`**: The capture clause. This is used to "capture" variables from the surrounding scope so they can be used inside the lambda. We will discuss this in the next section.
*   **`(parameters)`**: The parameter list, just like a regular function.
*   **`-> return_type`**: The return type. This is optional. If you omit it, the compiler will try to deduce the return type from the `return` statements in the function body.
*   **`{ function body }`**: The body of the lambda.

### Example: A simple lambda

```cpp
#include <iostream>

int main() {
    // A lambda that takes two integers and returns their sum
    auto my_lambda = [](int a, int b) -> int {
        return a + b;
    };

    int sum = my_lambda(3, 4); // call the lambda
    std::cout << "Sum: " << sum << std::endl;

    // You can also call a lambda immediately after defining it
    int product = [](int a, int b) { return a * b; }(5, 6);
    std::cout << "Product: " << product << std::endl;

    return 0;
}
```
In this example, the return type of the second lambda is deduced as `int`.

## Lambdas with STL Algorithms

The real power of lambdas comes from using them with STL algorithms. They allow you to write custom logic for algorithms like `std::sort`, `std::find_if`, and `std::for_each` in a very concise way.

### Example: `std::sort` with a custom comparator

Let's say we want to sort a vector of integers in descending order.

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> vec = {5, 2, 8, 1, 9};

    // Use a lambda as the comparison function
    std::sort(vec.begin(), vec.end(), [](int a, int b) {
        return a > b;
    });

    for (int x : vec) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

### Example: `std::find_if`

`std::find_if` finds the first element in a range that satisfies a certain condition. A lambda is a perfect way to specify this condition.

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5, 6};

    // Find the first element greater than 3
    auto it = std::find_if(vec.begin(), vec.end(), [](int x) {
        return x > 3;
    });

    if (it != vec.end()) {
        std::cout << "Found element: " << *it << std::endl;
    }

    return 0;
}
```
Before C++11, you would have had to define a separate function or a functor class to achieve the same result, which is much more verbose. Lambdas make the code more readable and expressive.
