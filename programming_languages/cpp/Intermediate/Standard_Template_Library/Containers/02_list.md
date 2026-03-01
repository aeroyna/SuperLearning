# `std::list`

`std::list` is a sequence container that represents a doubly-linked list. Unlike `std::vector`, the elements of a `std::list` are not stored in contiguous memory locations.

To use `std::list`, you need to include the `<list>` header.

### Visualization

```mermaid
graph LR
    Node1[Prev | Data: 10 | Next]
    Node2[Prev | Data: 20 | Next]
    Node3[Prev | Data: 30 | Next]
    
    Node1 -- Next --> Node2
    Node2 -- Prev --> Node1
    
    Node2 -- Next --> Node3
    Node3 -- Prev --> Node2
    
    style Node1 fill:#e1bee7
    style Node2 fill:#e1bee7
    style Node3 fill:#e1bee7
```


## `std::list` vs. `std::vector`

| `std::list`                                     | `std::vector`                                 |
|-------------------------------------------------|-----------------------------------------------|
| Fast insertion and deletion anywhere (O(1)).    | Slow insertion/deletion at the beginning/middle (O(n)). |
| Slow random access (O(n)).                      | Fast random access (O(1)).                    |
| Uses more memory due to pointers.               | More memory efficient.                        |

## Creating a List

```cpp
#include <list>

std::list<int> list1; // empty list
std::list<int> list2 = {1, 2, 3, 4, 5};
```

## Common Operations

Many of the operations for `std::list` are similar to `std::vector`, but there are some key differences.

### Adding Elements

*   `push_back()`: Adds an element to the end.
*   `push_front()`: Adds an element to the beginning.
*   `insert()`: Inserts an element at a specified position.

### Removing Elements

*   `pop_back()`: Removes the last element.
*   `pop_front()`: Removes the first element.
*   `erase()`: Removes an element or a range of elements.
*   `remove()`: Removes all elements with a specific value.

### Other useful methods

*   `sort()`: Sorts the list.
*   `reverse()`: Reverses the list.
*   `merge()`: Merges two sorted lists.
*   `unique()`: Removes consecutive duplicate elements.

### Example

```cpp
#include <iostream>
#include <list>

void print_list(const std::list<int>& l) {
    for (int x : l) {
        std::cout << x << " ";
    }
    std::cout << std::endl;
}

int main() {
    std::list<int> myList = {10, 20, 30, 40};

    myList.push_front(5);
    myList.push_back(50);
    print_list(myList);

    myList.pop_front();
    myList.pop_back();
    print_list(myList);

    myList.reverse();
    print_list(myList);

    return 0;
}
```

Because `std::list` does not provide random access, you cannot use the `[]` operator or `at()`. You must use iterators to access elements in the middle of the list.
