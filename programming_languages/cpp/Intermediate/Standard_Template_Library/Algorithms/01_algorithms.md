# Algorithms

The C++ Standard Library provides a rich set of functions for performing common operations on sequences of elements, such as sorting, searching, and counting. These are known as the **STL algorithms**.

To use these algorithms, you need to include the `<algorithm>` header.

The algorithms operate on ranges of elements, which are specified by a pair of iterators (usually `begin()` and `end()`).

## Non-modifying Sequence Operations

These algorithms do not modify the elements in the container.

*   `for_each()`: Applies a function to each element in a range.
*   `find()`: Finds the first occurrence of a value in a range.
*   `count()`: Counts the number of occurrences of a value in a range.
*   `equal()`: Compares two ranges for equality.
*   `search()`: Searches for a sub-sequence in a range.

### Example: `find` and `count`

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> vec = {10, 20, 30, 40, 20};

    // find
    auto it = std::find(vec.begin(), vec.end(), 30);
    if (it != vec.end()) {
        std::cout << "Found 30 at index: " << std::distance(vec.begin(), it) << std::endl;
    }

    // count
    int num_twenties = std::count(vec.begin(), vec.end(), 20);
    std::cout << "The number 20 appears " << num_twenties << " times." << std::endl;

    return 0;
}
```

## Modifying Sequence Operations

These algorithms can modify the elements in the container.

*   `copy()`: Copies a range of elements.
*   `move()`: Moves a range of elements.
*   `transform()`: Applies a function to a range of elements and stores the result in another range.
*   `replace()`: Replaces all occurrences of a value with another value.
*   `fill()`: Fills a range with a specific value.
*   `remove()`: Removes all elements with a specific value (note: this doesn't actually change the size of the container, it just moves the elements to be removed to the end).
*   `reverse()`: Reverses the order of elements in a range.
*   `random_shuffle()` / `shuffle()`: Randomly reorders the elements in a range.

## Sorting and Related Operations

*   `sort()`: Sorts a range of elements.
*   `stable_sort()`: Sorts a range, preserving the relative order of equal elements.
*   `binary_search()`: Searches for a value in a sorted range.
*   `merge()`: Merges two sorted ranges.
*   `min_element()`: Finds the smallest element in a range.
*   `max_element()`: Finds the largest element in a range.

### Example: `sort` and `binary_search`

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> vec = {5, 2, 8, 1, 9};

    std::sort(vec.begin(), vec.end());

    std::cout << "Sorted vector: ";
    for (int x : vec) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    if (std::binary_search(vec.begin(), vec.end(), 8)) {
        std::cout << "8 is in the vector." << std::endl;
    }

    return 0;
}
```

## Lambda Expressions with Algorithms

Lambda expressions (since C++11) are often used with STL algorithms to provide custom logic. For example, you can use a lambda to define a custom sorting criterion.

```cpp
// sort in descending order
std::sort(vec.begin(), vec.end(), [](int a, int b) {
    return a > b;
});
```
This is just a small sample of the available algorithms. The `<algorithm>` header is a powerful tool for writing concise and efficient C++ code.
