## Heap Fundamentals

While you typically won't have to implement a heap from scratch in an interview, understanding its underlying structure is important. The most common implementation is a **Binary Heap**, which uses a simple array to store data but represents it as a specialized binary tree.

### The Heap Property
A heap must satisfy a specific ordering constraint, known as the heap property.
- **Min-Heap Property**: For any given node `P`, its value is less than or equal to the values of its children `C`. `P.value <= C.value`.
- **Max-Heap Property**: The value of a parent node is greater than or equal to the values of its children. `P.value >= C.value`.

This property ensures that the smallest (in a min-heap) or largest (in a max-heap) element is always at the root of the tree.

### The Shape Property: A Complete Binary Tree
A binary heap must also be a **complete binary tree**. This means that all levels of the tree are completely filled, except possibly for the last level, which must be filled from left to right. This structural rule is what allows a heap to be stored compactly in an array without any gaps.

### Array-Based Representation
A binary heap can be visualized as a tree, but it is implemented as an array. The parent-child relationships are not stored with pointers, but are calculated using array indices.

For a node at index `i` in the array:
- Its **parent** is at index `(i - 1) // 2`.
- Its **left child** is at index `2 * i + 1`.
- Its **right child** is at index `2 * i + 2`.

This pointer-less representation is very space-efficient.

![Binary Heap Array Representation](https://media.geeksforgeeks.org/wp-content/uploads/20220707153724/CompleteBinaryTree.png)
*Source: GeeksForGeeks. A complete binary tree can be represented compactly in an array.*

When elements are added or removed, they are first placed at the end of the array to maintain the complete tree shape. Then, they are "bubbled up" or "sifted down" by swapping with parents/children until the heap property is restored. It is these swapping operations that give heaps their O(log n) complexity for insertions and deletions.
