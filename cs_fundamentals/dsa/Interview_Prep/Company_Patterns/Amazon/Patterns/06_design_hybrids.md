# Amazon Design Hybrid Patterns

**Frequency**: 🟡 **Medium**

Amazon commonly features coding design questions, often requiring the implementation of custom data structures or classes with specific API methods. These test your ability to choose optimal underlying data structures for performance.

## Key Concepts
- **Composite Structures**: Combining Hash Maps with Linked Lists (LRU), Arrays (GetRandom), or Trees (Time-based Key-Value).
- **Time/Space Trade-offs**: Discussing and implementing solutions that balance performance needs.
- **API Design**: Creating clean and efficient interfaces for custom classes.

## Phase 1: Must-Do (Foundation)

Master these 10 problems to build a solid foundation.

| Problem | Difficulty | Key Concept |
| :--- | :--- | :--- |
| [LRU Cache](https://leetcode.com/problems/lru-cache/) | Medium | Hash Map + Doubly Linked List. |
| [Min Stack](https://leetcode.com/problems/min-stack/) | Medium | Two Stacks. |
| [Implement Trie (Prefix Tree)](https://leetcode.com/problems/implement-trie-prefix-tree/) | Medium | Tree with char branches. |
| [Insert Delete GetRandom O(1)](https://leetcode.com/problems/insert-delete-getrandom-o1/) | Medium | Hash Map + Array. |
| [Time Based Key-Value Store](https://leetcode.com/problems/time-based-key-value-store/) | Medium | Hash Map + Binary Search (for timestamps). |
| [Logger Rate Limiter](https://leetcode.com/problems/logger-rate-limiter/) | Easy | Hash Map with timestamps. |
| [Design Hit Counter](https://leetcode.com/problems/design-hit-counter/) | Medium | Queue or Fixed Array (Timestamp, Count). |
| [Design HashMap](https://leetcode.com/problems/design-hashmap/) | Easy | Array of Linked Lists. |
| [Design HashSet](https://leetcode.com/problems/design-hashset/) | Easy | Array of Linked Lists. |
| [Design Tic-Tac-Toe](https://leetcode.com/problems/design-tic-tac-toe/) | Medium | Array/Counter optimization. |

## Phase 2: Practice & Variants (Depth)

Tackle these 10 harder variations and common follow-ups.

| Problem | Difficulty | Key Concept |
| :--- | :--- | :--- |
| [LFU Cache](https://leetcode.com/problems/lfu-cache/) | Hard | Hash Map + Freq Map + Doubly Linked Lists. |
| [Design Search Autocomplete System](https://leetcode.com/problems/design-search-autocomplete-system/) | Hard | Trie + Priority Queue. |
| [Snapshot Array](https://leetcode.com/problems/snapshot-array/) | Medium | Binary Search on version history. |
| [Stock Price Fluctuation](https://leetcode.com/problems/stock-price-fluctuation/) | Medium | Hash Map + Heap/TreeMap. |
| [Flatten Nested List Iterator](https://leetcode.com/problems/flatten-nested-list-iterator/) | Medium | Stack / Recursion. |
| [Design Twitter](https://leetcode.com/problems/design-twitter/) | Medium | Hash Map + Heap (Merge K Sorted). |
| [All O`one Data Structure](https://leetcode.com/problems/all-oone-data-structure/) | Hard | Doubly Linked List of buckets + Hash Map. |
| [Design In-Memory File System](https://leetcode.com/problems/design-in-memory-file-system/) | Hard | Trie / Directory Tree. |
| [Design Circular Queue](https://leetcode.com/problems/design-circular-queue/) | Medium | Array implementation. |
| [Design Max Stack](https://leetcode.com/problems/max-stack/) | Hard | Doubly Linked List + Map. |
