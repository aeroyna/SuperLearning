# Google System Design Hybrid Patterns

**Frequency**: 📈 **Increasing**

Google loves "Design" questions that require implementing a class with specific API methods. These test your ability to choose the right data structures to optimize for read vs. write operations.

## Key Concepts
- **Composite Structures**: Combining Hash Maps with Linked Lists (LRU) or Heaps (Median).
- **Lazy Propagation**: Delaying updates until necessary.
- **Randomization**: Using random sampling for O(1) average time.
- **Binary Search on History**: Managing versions or time-series data.

## Phase 1: Must-Do (Foundation)

Master these 10 problems to build a solid foundation in Data Structure Design.

| Problem | Difficulty | Key Concept |
| :--- | :--- | :--- |
| [LRU Cache](https://leetcode.com/problems/lru-cache/) | Medium | Hash Map + Doubly Linked List. |
| [Min Stack](https://leetcode.com/problems/min-stack/) | Medium | Two Stacks. |
| [Implement Trie (Prefix Tree)](https://leetcode.com/problems/implement-trie-prefix-tree/) | Medium | Tree with char branches. |
| [Logger Rate Limiter](https://leetcode.com/problems/logger-rate-limiter/) | Easy | Hash Map with timestamps. |
| [Moving Average from Data Stream](https://leetcode.com/problems/moving-average-from-data-stream/) | Easy | Queue / Circular Buffer. |
| [Insert Delete GetRandom O(1)](https://leetcode.com/problems/insert-delete-getrandom-o1/) | Medium | Hash Map + Array. |
| [Flatten Nested List Iterator](https://leetcode.com/problems/flatten-nested-list-iterator/) | Medium | Stack / Recursion. |
| [Design Hit Counter](https://leetcode.com/problems/design-hit-counter/) | Medium | Queue or Fixed Array (Timestamp, Count). |
| [Design Tic-Tac-Toe](https://leetcode.com/problems/design-tic-tac-toe/) | Medium | Array/Counter optimization. |
| [Serialize and Deserialize Binary Tree](https://leetcode.com/problems/serialize-and-deserialize-binary-tree/) | Hard | BFS/DFS String conversion. |

## Phase 2: Practice & Variants (Depth)

Tackle these 10 harder variations.

| Problem | Difficulty | Key Concept |
| :--- | :--- | :--- |
| [Design Search Autocomplete System](https://leetcode.com/problems/design-search-autocomplete-system/) | Hard | Trie + Priority Queue. |
| [Snapshot Array](https://leetcode.com/problems/snapshot-array/) | Medium | Binary Search on version history. |
| [Stock Price Fluctuation](https://leetcode.com/problems/stock-price-fluctuation/) | Medium | Hash Map + Heap/TreeMap. |
| [LFU Cache](https://leetcode.com/problems/lfu-cache/) | Hard | Hash Map + Freq Map + Linked Lists. |
| [Time Based Key-Value Store](https://leetcode.com/problems/time-based-key-value-store/) | Medium | Hash Map + Binary Search. |
| [Design In-Memory File System](https://leetcode.com/problems/design-in-memory-file-system/) | Hard | Trie / Directory Tree. |
| [Range Sum Query 2D - Mutable](https://leetcode.com/problems/range-sum-query-2d-mutable/) | Hard | 2D Fenwick Tree / Quadtree. |
| [Design Twitter](https://leetcode.com/problems/design-twitter/) | Medium | Hash Map + Heap (Merge K Sorted). |
| [All O`one Data Structure](https://leetcode.com/problems/all-oone-data-structure/) | Hard | Doubly Linked List of buckets + Hash Map. |
| [Design Excel Sum Formula](https://leetcode.com/problems/design-excel-sum-formula/) | Hard | Graph (Topological Sort) or Recursive Eval. |