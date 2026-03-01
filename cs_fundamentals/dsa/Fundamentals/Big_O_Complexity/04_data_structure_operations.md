## Time Complexity of Common Data Structure Operations

This cheatsheet provides the average and worst-case time complexities for the most common operations on fundamental data structures. Understanding these is essential for analyzing the performance of an algorithm.

---

### Array-based Structures

| Data Structure      | Operation               | Average Case | Worst Case   | Notes                                                        |
| ------------------- | ----------------------- | :----------: | :----------: | ------------------------------------------------------------ |
| **Array (Static)**  | Access by Index         |     O(1)     |     O(1)     |                                                              |
|                     | Search (Linear)         |     O(n)     |     O(n)     |                                                              |
| **Dynamic Array**   | Access by Index         |     O(1)     |     O(1)     |                                                              |
| (Python `list`)     | Insert/Delete (End)     |    O(1)*     |     O(n)     | *Amortized O(1). Worst case occurs during resize.           |
|                     | Insert/Delete (Middle)  |     O(n)     |     O(n)     | Requires shifting subsequent elements.                       |

---

### Hash-based Structures

| Data Structure      | Operation               | Average Case | Worst Case   | Notes                                                        |
| ------------------- | ----------------------- | :----------: | :----------: | ------------------------------------------------------------ |
| **Hash Table**      | Search / Insert / Delete|     O(1)     |     O(n)     | Worst case occurs with excessive hash collisions.            |
| (Python `dict`/`set`)|                         |              |              | Assumes a good hash function.                                |

---

### Linked Structures

| Data Structure      | Operation               | Average Case | Worst Case   | Notes                                                        |
| ------------------- | ----------------------- | :----------: | :----------: | ------------------------------------------------------------ |
| **Singly Linked List**| Access / Search         |     O(n)     |     O(n)     | Requires traversal from the head.                            |
|                     | Insert/Delete (Head)    |     O(1)     |     O(1)     |                                                              |
|                     | Insert/Delete (End)     |     O(n)     |     O(n)     | Requires traversal to find the tail.                         |
| **Doubly Linked List**| Insert/Delete (End)     |     O(1)     |     O(1)     | Assumes a tail pointer is maintained.                        |

---

### Tree-based Structures

| Data Structure      | Operation               | Average Case | Worst Case   | Notes                                                        |
| ------------------- | ----------------------- | :----------: | :----------: | ------------------------------------------------------------ |
| **Binary Search Tree**| Search / Insert / Delete|   O(log n)   |     O(n)     | Worst case occurs on an unbalanced (skewed) tree.            |
| **Balanced BST**    | Search / Insert / Delete|   O(log n)   |   O(log n)   | e.g., AVL Tree, Red-Black Tree.                              |
| (AVL, Red-Black)    |                         |              |              |                                                              |
| **Heap (Priority Q)**| Find Min/Max            |     O(1)     |     O(1)     |                                                              |
|                     | Insert / Delete Min/Max |   O(log n)   |   O(log n)   |                                                              |
|                     | Build Heap (`heapify`)  |     O(n)     |     O(n)     | A linear time operation.                                     |
| **Trie**            | Search / Insert         |     O(L)     |     O(L)     | L = length of the word. Independent of number of words.      |

---

### Queues & Stacks

| Data Structure      | Operation               | Time Complexity | Implementation Notes                                     |
| ------------------- | ----------------------- | :-------------: | -------------------------------------------------------- |
| **Stack**           | Push / Pop / Peek       |      O(1)       | Typically implemented with a dynamic array or linked list. |
| **Queue**           | Enqueue / Dequeue / Peek|      O(1)       | Requires an efficient implementation like a `deque` or linked list. |
