# Linked Lists

A linked list is a linear data structure where each element is a separate object, called a **node**. Each node contains some data and a pointer to the next node in the sequence.

Unlike arrays, linked lists are not stored in contiguous memory locations.

## Types of Linked Lists

*   **Singly Linked List:** Each node points only to the next node.
*   **Doubly Linked List:** Each node points to both the next node and the previous node. `std::list` is a doubly linked list.
*   **Circular Linked List:** The last node points back to the first node.

## Singly Linked List Implementation

Let's implement a simple singly linked list in C++.

### The `Node` struct

First, we need a `Node` struct.

```cpp
template <typename T>
struct Node {
    T data;
    std::unique_ptr<Node<T>> next;

    Node(T d) : data(d), next(nullptr) {}
};
```
We use `std::unique_ptr` to manage the memory of the nodes automatically.

### The `LinkedList` class

Now, we can create a `LinkedList` class that manages the nodes.

```cpp
#include <iostream>
#include <memory>

// Node struct as defined above...

template <typename T>
class LinkedList {
private:
    std::unique_ptr<Node<T>> head;

public:
    LinkedList() : head(nullptr) {}

    // Add a node to the front of the list
    void push_front(T data) {
        auto new_node = std::make_unique<Node<T>>(data);
        new_node->next = std::move(head);
        head = std::move(new_node);
    }

    // Print the list
    void print() const {
        Node<T>* current = head.get();
        while (current) {
            std::cout << current->data << " -> ";
            current = current->next.get();
        }
        std::cout << "nullptr" << std::endl;
    }
};

int main() {
    LinkedList<int> list;
    list.push_front(3);
    list.push_front(2);
    list.push_front(1);
    list.print(); // 1 -> 2 -> 3 -> nullptr
    return 0;
}
```

## Analysis

| Operation        | Average Case | Worst Case   |
|------------------|--------------|--------------|
| **Access**       | O(n)         | O(n)         |
| **Search**       | O(n)         | O(n)         |
| **Insertion**    | O(1)         | O(1)         |
| **Deletion**     | O(1)         | O(1)         |

*Note: Insertion and deletion are O(1) if you already have a pointer to the node before the insertion/deletion point. If you have to search for the node first, it becomes O(n).*

## Linked Lists vs. Arrays/Vectors

| Feature          | Arrays / `std::vector` | Linked Lists / `std::list` |
|------------------|------------------------|--------------------------|
| **Random Access**| O(1)                   | O(n)                     |
| **Insertion/Deletion (at end)** | O(1) (amortized)       | O(1)                     |
| **Insertion/Deletion (in middle)** | O(n)                   | O(1)                     |
| **Memory**       | Contiguous             | Non-contiguous           |
| **Overhead**     | Low                    | High (pointers)          |

In general, `std::vector` is usually the better choice for a general-purpose sequence container due to better cache performance (from contiguous memory) and fast random access. Linked lists are useful in specific scenarios where you have a large number of insertions and deletions in the middle of the sequence.
