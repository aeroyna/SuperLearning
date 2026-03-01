# Practice Problems: Modern C++ Features

## Problem 1: Structured Bindings (C++17)

You have a `std::map` of student names to their scores. Use a range-based `for` loop and structured bindings to print the name and score of each student.

### Solution

```cpp
#include <iostream>
#include <map>
#include <string>

int main() {
    std::map<std::string, int> student_scores = {
        {"Alice", 95},
        {"Bob", 88},
        {"Charlie", 92}
    };

    for (const auto& [name, score] : student_scores) {
        std::cout << name << " scored " << score << std::endl;
    }

    return 0;
}
```

## Problem 2: `if` with initializer (C++17)

Write a function that takes a `std::map<int, std::string>&` and an `int` key. Use `if` with an initializer to check if the key exists in the map. If it does, print the corresponding value.

### Solution

```cpp
#include <iostream>
#include <map>
#include <string>

void print_value_if_exists(const std::map<int, std::string>& m, int key) {
    if (auto it = m.find(key); it != m.end()) {
        std::cout << "Value for key " << key << " is: " << it->second << std::endl;
    } else {
        std::cout << "Key " << key << " not found." << std::endl;
    }
}

int main() {
    std::map<int, std::string> my_map = {{1, "one"}, {2, "two"}};
    print_value_if_exists(my_map, 1);
    print_value_if_exists(my_map, 3);
    return 0;
}
```

## Problem 3: `std::optional` (C++17)

Write a function `find_user` that takes a user ID and returns a `std::optional<std::string>` containing the username if the user is found, and an empty `std::optional` otherwise.

### Solution

```cpp
#include <iostream>
#include <string>
#include <optional>
#include <map>

// A mock database of users
std::map<int, std::string> user_database = {
    {101, "Alice"},
    {102, "Bob"}
};

std::optional<std::string> find_user(int user_id) {
    if (auto it = user_database.find(user_id); it != user_database.end()) {
        return it->second;
    }
    return std::nullopt; // or just {}
}

int main() {
    auto user1 = find_user(101);
    if (user1) { // or user1.has_value()
        std::cout << "Found user: " << *user1 << std::endl; // or user1.value()
    }

    auto user2 = find_user(103);
    if (!user2) {
        std::cout << "User 103 not found." << std::endl;
    }

    return 0;
}
```

## Problem 4: Ranges (C++20)

Given a `std::vector` of integers, use the C++20 ranges library to create a view that contains the squares of the even numbers, and then print the elements of the view.

### Solution

```cpp
#include <iostream>
#include <vector>
#include <ranges>

int main() {
    std::vector<int> numbers = {1, 2, 3, 4, 5, 6, 7, 8};

    auto even_squares = numbers
                      | std::views::filter([](int n) { return n % 2 == 0; })
                      | std::views::transform([](int n) { return n * n; });

    std::cout << "Squares of even numbers: ";
    for (int n : even_squares) {
        std::cout << n << " ";
    }
    std::cout << std::endl;

    return 0;
}
```
*Note: To compile this code, you will need a C++20 compliant compiler (e.g., g++ 10 or later) and you may need to link the ranges library if it's not part of the standard library distribution yet. Use the `-std=c++20` flag.*
