# Hash Tables

A hash table is a data structure that implements an associative array abstract data type, a structure that can map keys to values. `std::unordered_map` and `std::unordered_set` are implemented using hash tables.

## How it works

A hash table uses a **hash function** to compute an index into an array of **buckets** or **slots**, from which the desired value can be found.

1.  **Hash Function:** A good hash function should map the keys as evenly as possible over the array of buckets. For a given key, the hash function should always produce the same hash code.
2.  **Array of Buckets:** This is the underlying storage for the hash table.
3.  **Collision Resolution:** A collision occurs when two different keys hash to the same bucket. There are two main ways to handle collisions:

    *   **Separate Chaining:** Each bucket is a linked list (or other data structure) of key-value pairs. When a collision occurs, the new key-value pair is added to the list. This is the most common method.

    *   **Open Addressing:** All key-value pairs are stored in the bucket array itself. When a collision occurs, the hash table "probes" for the next available bucket. (e.g., linear probing, quadratic probing, double hashing).

### Example: Separate Chaining

Imagine a hash table with 10 buckets, and a hash function `hash(key) = key % 10`.

*   `insert(12)`: `hash(12) = 2`. Insert 12 into bucket 2.
*   `insert(22)`: `hash(22) = 2`. Collision! Add 22 to the linked list in bucket 2.
*   `insert(3)`: `hash(3) = 3`. Insert 3 into bucket 3.

## Analysis

| Operation   | Average Case | Worst Case |
|-------------|--------------|------------|
| **Search**  | O(1)         | O(n)       |
| **Insertion**| O(1)         | O(n)       |
| **Deletion** | O(1)         | O(n)       |

*   **Average Case:** The average case performance is O(1) if the hash function distributes the keys evenly.
*   **Worst Case:** The worst case performance is O(n) if all the keys hash to the same bucket. This can happen if the hash function is poor or if the hash table is too small.

To maintain good performance, a hash table will automatically **rehash** when the number of elements exceeds a certain threshold (the **load factor**). Rehashing involves creating a larger array of buckets and re-inserting all the elements.

## Hash Tables in C++

*   **`std::unordered_map`:** A hash table implementation of a map (key-value pairs).
*   **`std::unordered_set`:** A hash table implementation of a set (unique keys).

To use your own custom class as a key in `std::unordered_map` or `std::unordered_set`, you need to provide two things:
1.  A hash function for your class.
2.  An equality operator (`operator==`) for your class.

You can provide the hash function by specializing the `std::hash` template.

### Example: Custom Key

```cpp
#include <iostream>
#include <unordered_map>
#include <string>

struct Person {
    std::string first_name;
    std::string last_name;

    bool operator==(const Person& other) const {
        return first_name == other.first_name && last_name == other.last_name;
    }
};

// Specialize std::hash for Person
namespace std {
    template <>
    struct hash<Person> {
        size_t operator()(const Person& p) const {
            // A simple hash function
            return hash<string>()(p.first_name) ^ (hash<string>()(p.last_name) << 1);
        }
    };
}

int main() {
    std::unordered_map<Person, int> person_ages;
    person_ages[{ "John", "Doe" }] = 42;
    std::cout << person_ages[{ "John", "Doe" }] << std::endl;
    return 0;
}
```
