## Heaps and Priority Queues

A **Priority Queue** is an abstract data type that is similar to a regular queue but assigns a "priority" to each element. When you extract an element from a priority queue, you always get the element with the highest priority. In interviews, "highest priority" usually means the smallest or largest element.

The most common and efficient implementation of a priority queue is a **Heap**.

### Core Idea
A heap is a specialized tree-based data structure that satisfies the **heap property**.
- **Min-Heap**: The value of each node is less than or equal to the value of its children. This ensures that the node with the minimum value is always at the root.
- **Max-Heap**: The value of each node is greater than or equal to the value of its children. The maximum value is always at the root.

This structure allows a heap to provide the minimum or maximum element in O(1) time, while insertions and deletions are handled in O(log n) time.

### Analogy
Think of a hospital's emergency room triage. Patients are not seen in the order they arrive (FIFO). Instead, they are seen based on the severity of their condition (priority). A patient with a critical injury is treated before someone with a minor cut. The triage system acts as a priority queue, ensuring the highest priority patient is always next.

### Key Characteristics
- **Fast Min/Max Access**: O(1) to find the minimum (in a min-heap) or maximum (in a max-heap) element.
- **Efficient Updates**: O(log n) time to add a new element or remove the min/max element, while maintaining the heap property.
- **Not for Searching**: Heaps are not designed for fast searching of arbitrary elements. Checking for the existence of an element takes O(n) time. If you need fast search, a hash set or BST is a better choice.

### Practice
- [Practice Problems](Practice_Problems/00_practice_problems.md)