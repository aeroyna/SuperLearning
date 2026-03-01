# Algorithms

Algorithms are systematic procedures for solving computational problems. This section covers the core algorithmic paradigms essential for coding interviews: searching, sorting, dynamic programming, graph algorithms, and optimization techniques.

## Overview

Algorithm categories by problem-solving approach:
- **Divide and Conquer**: Break problem into subproblems (Binary Search, Merge Sort)
- **Dynamic Programming**: Solve overlapping subproblems optimally
- **Greedy**: Make locally optimal choices
- **Backtracking**: Explore all possibilities systematically
- **Graph Algorithms**: Traverse and analyze graph structures

## Topics

### Searching
- [14. Binary Search](Binary_Search/00_binary_search.md) - O(log n) searching and beyond

### Sorting
- [13. Sorting Algorithms](Sorting/00_sorting.md) - Comparison and non-comparison sorts

### Optimization
- [16. Dynamic Programming](Dynamic_Programming/00_dynamic_programming.md) - Optimal substructure problems
- [17. Greedy Algorithms](Greedy_Algorithms/00_greedy_algorithms.md) - Local optimality approach

### Exploration
- [15. Recursion and Backtracking](Recursion_and_Backtracking/00_recursion_and_backtracking.md) - Systematic enumeration

### Graph Algorithms
- [18-20. Graph Algorithms](Graph_Algorithms/00_graph_algorithms.md) - Shortest paths, MST, topological sort

## Algorithm Selection Framework

```
What type of problem?
│
├── Searching
│   ├── Sorted data → Binary Search
│   ├── Unsorted data → Linear Search or Hash
│   └── Graph/tree traversal → BFS/DFS
│
├── Optimization (max/min)
│   ├── Overlapping subproblems? → Dynamic Programming
│   ├── Greedy choice property? → Greedy
│   └── Need all solutions? → Backtracking
│
├── Enumeration (generate all)
│   └── Backtracking with pruning
│
└── Graph problems
    ├── Shortest path (unweighted) → BFS
    ├── Shortest path (weighted, non-negative) → Dijkstra
    ├── Shortest path (negative weights) → Bellman-Ford
    ├── Minimum spanning tree → Kruskal/Prim
    └── Dependency ordering → Topological Sort
```

## Complexity Classes

| Algorithm | Time | Space | Key Insight |
|-----------|------|-------|-------------|
| Binary Search | O(log n) | O(1) | Halve search space each step |
| Merge Sort | O(n log n) | O(n) | Divide, sort, merge |
| Quick Sort | O(n log n) avg | O(log n) | Partition around pivot |
| Dijkstra | O((V+E) log V) | O(V) | Greedy shortest path |
| DP (typical) | O(states × transitions) | O(states) | Avoid recomputation |

## Key Patterns by Frequency

### Most Common in Interviews
1. Binary Search (and its variants)
2. Two Pointers / Sliding Window
3. BFS/DFS
4. Dynamic Programming
5. Backtracking

### Important but Less Common
6. Topological Sort
7. Union-Find
8. Dijkstra's Algorithm
9. Sorting algorithms (implementation)

## Common Pitfalls

1. **Wrong algorithm choice**: Greedy when DP needed, or vice versa
2. **Missing edge cases**: Empty input, single element, all same elements
3. **Integer overflow**: In sum calculations, especially DP
4. **Infinite loops**: In binary search (incorrect bounds update)
5. **Stack overflow**: Deep recursion without memoization

## Study Order Recommendation

1. **Week 1-2**: Binary Search, Two Pointers, Sliding Window
2. **Week 3-4**: BFS/DFS, Basic DP
3. **Week 5-6**: Backtracking, Advanced DP patterns
4. **Week 7-8**: Graph algorithms, Greedy
