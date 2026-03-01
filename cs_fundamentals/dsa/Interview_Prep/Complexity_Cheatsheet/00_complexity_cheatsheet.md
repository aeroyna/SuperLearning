# Complexity Cheatsheet

A quick reference for time and space complexity of common data structure operations and algorithms. Use this to quickly verify your complexity analysis during interviews.

## Overview

This cheatsheet covers:
- Data structure operation complexities
- Algorithm complexities
- Space complexity patterns
- Input size guidelines

## Topics

- [26.1 Data Structure Operations](01_data_structure_operations.md)
- [26.2 Algorithm Complexities](02_algorithm_complexities.md)
- [26.3 Space Complexity Guide](03_space_complexity_guide.md)

## Data Structure Operations

### Array / List

| Operation | Time | Notes |
|-----------|------|-------|
| Access by index | O(1) | |
| Search (unsorted) | O(n) | |
| Search (sorted) | O(log n) | Binary search |
| Insert at end | O(1)* | Amortized |
| Insert at beginning | O(n) | Shift all elements |
| Insert at middle | O(n) | |
| Delete at end | O(1) | |
| Delete at beginning | O(n) | |
| Delete at middle | O(n) | |

### Hash Table

| Operation | Average | Worst |
|-----------|---------|-------|
| Insert | O(1) | O(n) |
| Delete | O(1) | O(n) |
| Search | O(1) | O(n) |

### Linked List

| Operation | Singly | Doubly |
|-----------|--------|--------|
| Access | O(n) | O(n) |
| Search | O(n) | O(n) |
| Insert at head | O(1) | O(1) |
| Insert at tail | O(n)* | O(1) |
| Delete given node | O(n) | O(1) |

*O(1) with tail pointer

### Stack / Queue

| Operation | Time |
|-----------|------|
| Push/Enqueue | O(1) |
| Pop/Dequeue | O(1) |
| Peek | O(1) |

### Binary Heap

| Operation | Time |
|-----------|------|
| Insert | O(log n) |
| Extract min/max | O(log n) |
| Peek min/max | O(1) |
| Build heap | O(n) |
| Search | O(n) |

### Binary Search Tree (Balanced)

| Operation | Average | Worst (Unbalanced) |
|-----------|---------|-------------------|
| Search | O(log n) | O(n) |
| Insert | O(log n) | O(n) |
| Delete | O(log n) | O(n) |
| Min/Max | O(log n) | O(n) |

### Trie

| Operation | Time | Notes |
|-----------|------|-------|
| Insert | O(m) | m = key length |
| Search | O(m) | |
| Prefix search | O(m) | |

## Algorithm Complexities

### Sorting

| Algorithm | Best | Average | Worst | Space | Stable |
|-----------|------|---------|-------|-------|--------|
| Bubble Sort | O(n) | O(n²) | O(n²) | O(1) | Yes |
| Selection Sort | O(n²) | O(n²) | O(n²) | O(1) | No |
| Insertion Sort | O(n) | O(n²) | O(n²) | O(1) | Yes |
| Merge Sort | O(n log n) | O(n log n) | O(n log n) | O(n) | Yes |
| Quick Sort | O(n log n) | O(n log n) | O(n²) | O(log n) | No |
| Heap Sort | O(n log n) | O(n log n) | O(n log n) | O(1) | No |
| Counting Sort | O(n+k) | O(n+k) | O(n+k) | O(k) | Yes |
| Radix Sort | O(nk) | O(nk) | O(nk) | O(n+k) | Yes |

### Graph Algorithms

| Algorithm | Time | Space |
|-----------|------|-------|
| BFS | O(V + E) | O(V) |
| DFS | O(V + E) | O(V) |
| Dijkstra (heap) | O((V+E) log V) | O(V) |
| Bellman-Ford | O(VE) | O(V) |
| Floyd-Warshall | O(V³) | O(V²) |
| Kruskal | O(E log E) | O(V) |
| Prim (heap) | O(E log V) | O(V) |
| Topological Sort | O(V + E) | O(V) |

### Tree Algorithms

| Algorithm | Time | Space |
|-----------|------|-------|
| Tree traversal | O(n) | O(h) |
| BST search | O(log n)* | O(1) |
| BST insert | O(log n)* | O(1) |
| Segment tree query | O(log n) | O(1) |
| Segment tree update | O(log n) | O(1) |

*Balanced tree

### String Algorithms

| Algorithm | Time | Space |
|-----------|------|-------|
| Naive pattern match | O(nm) | O(1) |
| KMP | O(n + m) | O(m) |
| Rabin-Karp (avg) | O(n + m) | O(1) |

## Input Size Guidelines

Use problem constraints to infer expected complexity:

| Input Size | Expected Complexity | Typical Approach |
|------------|--------------------|--------------------|
| n ≤ 10 | O(n!) or O(2^n) | Brute force, backtracking |
| n ≤ 20 | O(2^n) | Bitmask DP, backtracking |
| n ≤ 100 | O(n³) | 3 nested loops, Floyd-Warshall |
| n ≤ 1000 | O(n²) | 2D DP, nested loops |
| n ≤ 10^4 | O(n√n) or O(n log n) | Sorting, binary search |
| n ≤ 10^5 | O(n log n) | Sorting, heap, tree |
| n ≤ 10^6 | O(n) | Hash map, two pointers |
| n ≤ 10^7 | O(n) | Linear scan, streaming |
| n > 10^7 | O(log n) or O(1) | Binary search, math |

## Space Complexity Patterns

| Pattern | Space | Example |
|---------|-------|---------|
| In-place | O(1) | Two pointers, swapping |
| Linear scan | O(1) | Iteration without storage |
| Hash set/map | O(n) | Storing elements |
| Recursion (tail) | O(h) | DFS, where h = depth |
| Recursion + memo | O(states) | DP memoization |
| Matrix storage | O(n²) | 2D DP |
| Adjacency list | O(V + E) | Graph storage |

## Quick Lookup

### Common Time Complexities (Best to Worst)

```
O(1) < O(log n) < O(n) < O(n log n) < O(n²) < O(n³) < O(2^n) < O(n!)
```

### Operations Per Second

Rough estimate: ~10^8 operations per second

| Complexity | n = 10^6 | Operations |
|------------|----------|------------|
| O(n) | 10^6 | ~1 second |
| O(n log n) | ~2×10^7 | ~0.2 seconds |
| O(n²) | 10^12 | ~3 hours |

## Common Mistakes

1. **Forgetting amortized complexity**: Dynamic array append is O(1) amortized
2. **Ignoring hidden constants**: Hash operations have overhead
3. **Missing space for recursion**: Call stack counts as space
4. **Assuming sorted input**: Check if sorting is needed
5. **Best case vs worst case**: Report worst case unless asked
