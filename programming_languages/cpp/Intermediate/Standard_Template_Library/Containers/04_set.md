# `std::set`

`std::set` is an associative container that stores a collection of unique elements. The elements are always sorted.

Like `std::map`, `std::set` is usually implemented as a red-black tree, providing fast search, insertion, and deletion (O(log n)).

To use `std::set`, you need to include the `<set>` header.

## Creating a Set

```cpp
#include <set>

std::set<int> my_set;
std::set<int> my_set2 = {1, 5, 2, 4, 3, 1, 2}; // will contain {1, 2, 3, 4, 5}
```

## Common Operations

### Inserting Elements

*   `insert()`: Inserts an element into the set. If the element is already in the set, the set is not modified.

    ```cpp
    my_set.insert(10);
    my_set.insert(20);
    my_set.insert(10); // no effect
    ```

### Removing Elements

*   `erase()`: Removes an element by value, by position (iterator), or a range of elements.

    ```cpp
    my_set.erase(20);
    ```

### Searching for Elements

*   `find()`: Searches for an element. Returns an iterator to the element if found, or an iterator to the end of the set if not found.

    ```cpp
    auto it = my_set.find(10);
    if (it != my_set.end()) {
        std::cout << "Found 10" << std::endl;
    }
    ```

*   `count()`: Returns 1 if the element is in the set, and 0 otherwise.

    ```cpp
    if (my_set.count(10)) {
        std::cout << "10 is in the set." << std::endl;
    }
    ```

### Iterating through a Set

You can iterate through a set using a range-based for loop.

```cpp
for (int x : my_set) {
    std::cout << x << " ";
}
```

### Example

```cpp
#include <iostream>
#include <set>
#include <string>

int main() {
    std::set<std::string> unique_words;

    unique_words.insert("apple");
    unique_words.insert("banana");
    unique_words.insert("apple");
    unique_words.insert("orange");

    std::cout << "Unique words: ";
    for (const std::string& word : unique_words) {
        std::cout << word << " ";
    }
    std::cout << std::endl;

    if (unique_words.count("banana")) {
        std::cout << "The set contains 'banana'" << std::endl;
    }

    return 0;
}
```

## `std::unordered_set` (C++11 and later)

Similar to `std::unordered_map`, `std::unordered_set` is a version of `std::set` that is implemented using a hash table. It provides average case O(1) performance for insertion, deletion, and search, but the elements are not sorted.
