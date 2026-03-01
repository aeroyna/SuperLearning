# Complexity Cheatsheet

A quick reference for time and space complexity of common operations.

## Data Structure Operations

### Arrays

```
┌─────────────────────────────────────────────────────────┐
│ Array (Dynamic)                                         │
├─────────────────────┬───────────┬───────────────────────┤
│ Operation           │ Average   │ Worst                 │
├─────────────────────┼───────────┼───────────────────────┤
│ Access by index     │ O(1)      │ O(1)                  │
│ Search              │ O(n)      │ O(n)                  │
│ Insert at end       │ O(1)*     │ O(n) - resize         │
│ Insert at beginning │ O(n)      │ O(n)                  │
│ Delete at end       │ O(1)      │ O(1)                  │
│ Delete at beginning │ O(n)      │ O(n)                  │
└─────────────────────┴───────────┴───────────────────────┘
* amortized
```

### Linked Lists

```
┌─────────────────────────────────────────────────────────┐
│ Linked List (Singly)                                    │
├─────────────────────┬───────────┬───────────────────────┤
│ Operation           │ Average   │ Worst                 │
├─────────────────────┼───────────┼───────────────────────┤
│ Access by index     │ O(n)      │ O(n)                  │
│ Search              │ O(n)      │ O(n)                  │
│ Insert at head      │ O(1)      │ O(1)                  │
│ Insert at tail      │ O(n)      │ O(n) (O(1) if tracked)│
│ Delete at head      │ O(1)      │ O(1)                  │
│ Delete at tail      │ O(n)      │ O(n)                  │
└─────────────────────┴───────────┴───────────────────────┘
```

### Hash Table

```
┌─────────────────────────────────────────────────────────┐
│ Hash Table                                              │
├─────────────────────┬───────────┬───────────────────────┤
│ Operation           │ Average   │ Worst                 │
├─────────────────────┼───────────┼───────────────────────┤
│ Search              │ O(1)      │ O(n)                  │
│ Insert              │ O(1)      │ O(n)                  │
│ Delete              │ O(1)      │ O(n)                  │
└─────────────────────┴───────────┴───────────────────────┘
```

### Binary Search Tree

```
┌─────────────────────────────────────────────────────────┐
│ BST                                                     │
├─────────────────────┬───────────┬───────────────────────┤
│ Operation           │ Average   │ Worst (unbalanced)    │
├─────────────────────┼───────────┼───────────────────────┤
│ Search              │ O(log n)  │ O(n)                  │
│ Insert              │ O(log n)  │ O(n)                  │
│ Delete              │ O(log n)  │ O(n)                  │
│ Find Min/Max        │ O(log n)  │ O(n)                  │
└─────────────────────┴───────────┴───────────────────────┘
```

### Balanced BST (AVL, Red-Black)

```
┌─────────────────────────────────────────────────────────┐
│ Balanced BST                                            │
├─────────────────────┬───────────┬───────────────────────┤
│ Operation           │ Average   │ Worst                 │
├─────────────────────┼───────────┼───────────────────────┤
│ Search              │ O(log n)  │ O(log n)              │
│ Insert              │ O(log n)  │ O(log n)              │
│ Delete              │ O(log n)  │ O(log n)              │
└─────────────────────┴───────────┴───────────────────────┘
```

### Heap (Binary)

```
┌─────────────────────────────────────────────────────────┐
│ Binary Heap                                             │
├─────────────────────┬───────────────────────────────────┤
│ Operation           │ Time Complexity                   │
├─────────────────────┼───────────────────────────────────┤
│ Find Min/Max        │ O(1)                              │
│ Insert              │ O(log n)                          │
│ Extract Min/Max     │ O(log n)                          │
│ Heapify (build)     │ O(n)                              │
│ Search              │ O(n)                              │
└─────────────────────┴───────────────────────────────────┘
```

## Sorting Algorithms

```
┌────────────────────────────────────────────────────────────────────────┐
│ Sorting Algorithms                                                      │
├─────────────────┬─────────────┬─────────────┬─────────────┬────────────┤
│ Algorithm       │ Best        │ Average     │ Worst       │ Space      │
├─────────────────┼─────────────┼─────────────┼─────────────┼────────────┤
│ Bubble Sort     │ O(n)        │ O(n²)       │ O(n²)       │ O(1)       │
│ Selection Sort  │ O(n²)       │ O(n²)       │ O(n²)       │ O(1)       │
│ Insertion Sort  │ O(n)        │ O(n²)       │ O(n²)       │ O(1)       │
│ Merge Sort      │ O(n log n)  │ O(n log n)  │ O(n log n)  │ O(n)       │
│ Quick Sort      │ O(n log n)  │ O(n log n)  │ O(n²)       │ O(log n)   │
│ Heap Sort       │ O(n log n)  │ O(n log n)  │ O(n log n)  │ O(1)       │
│ Counting Sort   │ O(n + k)    │ O(n + k)    │ O(n + k)    │ O(k)       │
│ Radix Sort      │ O(nk)       │ O(nk)       │ O(nk)       │ O(n + k)   │
│ Bucket Sort     │ O(n + k)    │ O(n + k)    │ O(n²)       │ O(n)       │
└─────────────────┴─────────────┴─────────────┴─────────────┴────────────┘
```

## Graph Algorithms

```
┌────────────────────────────────────────────────────────────────────────┐
│ Graph Algorithms (V = vertices, E = edges)                              │
├───────────────────────────┬───────────────────┬─────────────────────────┤
│ Algorithm                 │ Time              │ Space                   │
├───────────────────────────┼───────────────────┼─────────────────────────┤
│ DFS                       │ O(V + E)          │ O(V)                    │
│ BFS                       │ O(V + E)          │ O(V)                    │
│ Dijkstra (binary heap)    │ O((V + E) log V)  │ O(V)                    │
│ Bellman-Ford              │ O(V * E)          │ O(V)                    │
│ Floyd-Warshall            │ O(V³)             │ O(V²)                   │
│ Topological Sort          │ O(V + E)          │ O(V)                    │
│ Kruskal's MST             │ O(E log E)        │ O(V)                    │
│ Prim's MST                │ O((V + E) log V)  │ O(V)                    │
│ Union-Find (path comp.)   │ O(α(n)) ≈ O(1)    │ O(V)                    │
└───────────────────────────┴───────────────────┴─────────────────────────┘
```

## Common Patterns

```
┌────────────────────────────────────────────────────────────────────────┐
│ Common Patterns                                                         │
├───────────────────────────┬───────────────────┬─────────────────────────┤
│ Pattern                   │ Time              │ Space                   │
├───────────────────────────┼───────────────────┼─────────────────────────┤
│ Two Pointers              │ O(n)              │ O(1)                    │
│ Sliding Window            │ O(n)              │ O(1) to O(k)            │
│ Binary Search             │ O(log n)          │ O(1)                    │
│ Prefix Sum                │ O(n) build, O(1)  │ O(n)                    │
│ Monotonic Stack           │ O(n)              │ O(n)                    │
│ Backtracking              │ O(k^n) or O(n!)   │ O(n)                    │
│ DP (1D)                   │ O(n)              │ O(n) or O(1)            │
│ DP (2D)                   │ O(n*m)            │ O(n*m) or O(min(n,m))   │
└───────────────────────────┴───────────────────┴─────────────────────────┘
```

## DS/A Selection Flowchart

```
Need to store data and...
│
├─> Need O(1) access by index?
│   └─> Array
│
├─> Need O(1) lookup by key?
│   └─> Hash Map/Set
│
├─> Need sorted order + fast insert/delete?
│   └─> Balanced BST (TreeMap/TreeSet)
│
├─> Need min/max repeatedly?
│   └─> Heap
│
├─> Need LIFO access?
│   └─> Stack
│
├─> Need FIFO access?
│   └─> Queue
│
├─> Need frequent insert/delete in middle?
│   └─> Linked List
│
├─> Need prefix matching?
│   └─> Trie
│
└─> Need range queries/updates?
    └─> Segment Tree / Fenwick Tree
```

## Quick Memory Tricks

1. **Log n appears when**: Halving search space (binary search, balanced trees)
2. **n log n appears when**: Divide and conquer with linear merge (merge sort)
3. **n² appears when**: Nested loops over same input
4. **2^n appears when**: All subsets (take/don't take decisions)
5. **n! appears when**: All permutations
