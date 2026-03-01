# Iterators

An iterator is an object that points to an element in a container. Iterators are used to traverse the elements of containers like vectors, lists, maps, etc.

## Analogy to Pointers

Iterators are similar to pointers, but they are a more general concept. You can think of an iterator as a "smart pointer" for containers.

## Common Iterator Operations

*   `*it`: Dereference the iterator to get the value of the element it points to.
*   `it++`: Move the iterator to the next element.
*   `it == other_it`: Compare two iterators for equality.
*   `it != other_it`: Compare two iterators for inequality.

## Getting Iterators from Containers

Most containers provide member functions to get iterators:

*   `begin()`: Returns an iterator to the first element.
*   `end()`: Returns an iterator to the "past-the-end" element. This is a placeholder after the last element and should not be dereferenced.
*   `rbegin()`: Returns a reverse iterator to the last element.
*   `rend()`: Returns a reverse iterator to the theoretical element preceding the first element.

### Example with `std::vector`

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};

    // Declare an iterator
    std::vector<int>::iterator it;

    // Traverse the vector using iterators
    for (it = vec.begin(); it != vec.end(); ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;

    // Modify elements using an iterator
    for (it = vec.begin(); it != vec.end(); ++it) {
        *it = *it * 2;
    }

    // Print the modified vector
    for (int x : vec) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

## `auto` Keyword with Iterators

The `auto` keyword (since C++11) makes working with iterators much easier, as you don't have to write out the long type names.

```cpp
for (auto it = vec.begin(); it != vec.end(); ++it) {
    // ...
}
```

## Iterator Categories

There are different categories of iterators, with different levels of functionality:

1.  **Input Iterators:** Can be used to read elements from a container. Can only be incremented.
2.  **Output Iterators:** Can be used to write elements to a container. Can only be incremented.
3.  **Forward Iterators:** Combine the functionality of input and output iterators. Can be incremented multiple times. (`std::forward_list`)
4.  **Bidirectional Iterators:** Like forward iterators, but can also be decremented. (`std::list`, `std::map`, `std::set`)
5.  **Random Access Iterators:** Like bidirectional iterators, but can also be moved by any amount in constant time (e.g., `it + 5`). (`std::vector`, `std::deque`, `std::array`)
