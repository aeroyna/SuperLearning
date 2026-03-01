## Binary Search Trees (BST)

A Binary Search Tree is a node-based binary tree data structure which has the following key properties:
- The left subtree of a node contains only nodes with keys lesser than the node’s key.
- The right subtree of a node contains only nodes with keys greater than the node’s key.
- The left and right subtree each must also be a binary search tree.
- There must be no duplicate keys.

This ordering property is the defining feature of a BST and allows for highly efficient search, insertion, and deletion operations.

### Core Idea & Analogy
The BST property allows you to perform a binary search on a tree structure. Think of a "Guess the Number" game. With each guess, you're told "higher" or "lower". This is exactly how you traverse a BST. At any node, comparing your target value to the node's value tells you whether to go left (for a smaller value) or right (for a larger value), effectively cutting the remaining search space in half at each step.

This leads to an average time complexity of O(log n) for most operations, which is significantly faster than the O(n) required for an unsorted data structure.

### Key Characteristics
- **Efficiency**: O(log n) average time for search, insert, and delete.
- **Ordered Data**: A BST naturally keeps its elements in a sorted order. An in-order traversal of a BST will always yield the elements in ascending order.
- **Worst-Case Scenario**: If the tree becomes unbalanced (e.g., by inserting elements in sorted order), it can degenerate into a structure resembling a linked list, making the time complexity for operations degrade to O(n). Self-balancing BSTs (like AVL or Red-Black Trees) are used in practice to prevent this, but are generally considered an advanced topic for interviews.


### Practice
- [Practice Problems](Practice_Problems/00_practice_problems.md)