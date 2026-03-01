# DS/A Selection Flowchart

Use this flowchart to quickly identify which data structure or algorithm to use for a given problem.

## Quick Decision Tree

```
START: What does the problem ask for?
│
├─> Finding/Searching?
│   ├─> Sorted data? → Binary Search
│   ├─> By key/value? → Hash Map/Set
│   └─> In graph/tree? → BFS/DFS
│
├─> Optimal value (max/min)?
│   ├─> Overlapping subproblems? → Dynamic Programming
│   ├─> Local choice = global optimal? → Greedy
│   └─> Need to try all possibilities? → Backtracking
│
├─> Subarray/Substring?
│   ├─> With constraint? → Sliding Window
│   ├─> Sum in range? → Prefix Sum
│   └─> Find pair? → Two Pointers
│
├─> Order/Sequence?
│   ├─> LIFO (last in, first out)? → Stack
│   ├─> FIFO (first in, first out)? → Queue
│   └─> By priority? → Heap
│
├─> Connected components/paths?
│   ├─> Shortest path (unweighted)? → BFS
│   ├─> Shortest path (weighted)? → Dijkstra
│   └─> All possibilities? → DFS
│
└─> Top K elements?
    └─> Heap (size K)
```

## Pattern Recognition

### Array/String Problems

| If you see... | Think... |
|---------------|----------|
| "Sorted array" | Binary Search |
| "Subarray with property" | Sliding Window |
| "Sum of range" | Prefix Sum |
| "Find pair" | Two Pointers / Hash Map |
| "Contiguous elements" | Sliding Window |
| "Unique elements" | Hash Set |

### Tree/Graph Problems

| If you see... | Think... |
|---------------|----------|
| "Level by level" | BFS |
| "Path finding" | DFS/BFS |
| "Connected components" | DFS/Union-Find |
| "Shortest path" | BFS (unweighted) / Dijkstra (weighted) |
| "Cycle detection" | DFS with coloring |
| "Dependencies" | Topological Sort |

### Optimization Problems

| If you see... | Think... |
|---------------|----------|
| "Maximum/Minimum" | DP or Greedy |
| "Number of ways" | DP |
| "All combinations/permutations" | Backtracking |
| "Optimal substructure" | DP |
| "Local choice → global optimal" | Greedy |

### Special Patterns

| If you see... | Think... |
|---------------|----------|
| "K largest/smallest" | Heap |
| "Median" | Two Heaps |
| "Next greater/smaller" | Monotonic Stack |
| "Intervals" | Sort + Greedy |
| "Prefix matching" | Trie |
| "Range updates/queries" | Segment Tree / BIT |

## Complexity Hints

Based on input constraints:

| n ≤ | Expected Complexity | Typical Approach |
|-----|--------------------|--------------------|
| 10 | O(n!) | Brute force, backtracking |
| 20 | O(2^n) | Subsets, bitmask |
| 100 | O(n³) | Triple nested loops |
| 1,000 | O(n²) | Nested loops, 2D DP |
| 10^5 | O(n log n) | Sort + scan, binary search |
| 10^6 | O(n) | Hash, two pointers |
| 10^9+ | O(log n) | Binary search, math |

## Quick Reference by Problem Type

### "Find" Problems
- Single element: Hash Set
- Pair: Two Pointers / Hash Map
- K elements: Heap
- In sorted: Binary Search

### "Count" Problems
- Frequency: Hash Map
- Subarrays: Prefix Sum + Hash
- Ways: DP

### "Check/Validate" Problems
- Existence: Hash Set
- Balanced: Stack
- Valid: State machine / DP

### "Transform" Problems
- In-place: Two Pointers
- New array: Map operation
- String build: StringBuilder/List
