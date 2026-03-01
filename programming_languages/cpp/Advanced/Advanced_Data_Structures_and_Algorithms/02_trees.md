# Trees

A tree is a hierarchical data structure that consists of nodes connected by edges.

## Terminology

*   **Node:** An entity that contains a key or value and pointers to its child nodes.
*   **Edge:** The link between two nodes.
*   **Root:** The topmost node in a tree.
*   **Parent:** A node that has child nodes.
*   **Child:** A node that has a parent node.
*   **Leaf:** A node that does not have any child nodes.
*   **Height of a tree:** The number of edges on the longest path from the root to a leaf.
*   **Depth of a node:** The number of edges from the root to the node.

## Binary Tree

A binary tree is a tree where each node has at most two children, referred to as the left child and the right child.

## Binary Search Tree (BST)

A Binary Search Tree is a special type of binary tree with the following properties:
*   The left subtree of a node contains only nodes with keys lesser than the node's key.
*   The right subtree of a node contains only nodes with keys greater than the node's key.
*   The left and right subtree each must also be a binary search tree.
*   There must be no duplicate nodes.

### BST Implementation

```cpp
#include <iostream>
#include <memory>

template <typename T>
struct Node {
    T data;
    std::unique_ptr<Node<T>> left;
    std::unique_ptr<Node<T>> right;

    Node(T d) : data(d), left(nullptr), right(nullptr) {}
};

template <typename T>
class BinarySearchTree {
private:
    std::unique_ptr<Node<T>> root;

    void insert(std::unique_ptr<Node<T>>& node, T data) {
        if (!node) {
            node = std::make_unique<Node<T>>(data);
        } else if (data < node->data) {
            insert(node->left, data);
        } else if (data > node->data) {
            insert(node->right, data);
        }
    }

    void inorder_traversal(const std::unique_ptr<Node<T>>& node) const {
        if (node) {
            inorder_traversal(node->left);
            std::cout << node->data << " ";
            inorder_traversal(node->right);
        }
    }

public:
    BinarySearchTree() : root(nullptr) {}

    void insert(T data) {
        insert(root, data);
    }

    void print_inorder() const {
        inorder_traversal(root);
        std::cout << std::endl;
    }
};

int main() {
    BinarySearchTree<int> bst;
    bst.insert(50);
    bst.insert(30);
    bst.insert(70);
    bst.insert(20);
    bst.insert(40);
    
    bst.print_inorder(); // 20 30 40 50 70

    return 0;
}
```

### Analysis of BST

| Operation   | Average Case | Worst Case |
|-------------|--------------|------------|
| **Search**  | O(log n)     | O(n)       |
| **Insertion**| O(log n)     | O(n)       |
| **Deletion** | O(log n)     | O(n)       |

The worst-case performance occurs when the tree is unbalanced (e.g., a "degenerate" tree that looks like a linked list).

## Self-Balancing Binary Search Trees

To avoid the worst-case scenario, we can use self-balancing BSTs. These trees automatically keep their height small in the face of arbitrary insertions and deletions.

*   **AVL Tree:** One of the first self-balancing BSTs. It maintains a balance factor for each node and performs rotations to keep the tree balanced.
*   **Red-Black Tree:** Another type of self-balancing BST. `std::map` and `std::set` are typically implemented using red-black trees.

## Other Types of Trees

*   **Trie:** A tree-like data structure that is used for storing a dynamic set of strings.
*   **Heap:** A specialized tree-based data structure that satisfies the heap property. `std::priority_queue` is implemented as a heap.
*   **B-Tree:** A self-balancing tree that is optimized for systems that read and write large blocks of data. They are commonly used in databases and filesystems.
