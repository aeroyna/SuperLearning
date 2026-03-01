# Microsoft Design & Heaps Patterns

**Frequency**: 🟡 **Medium/High**

Microsoft asks system design coding questions (OO design) and heap-based problems frequently. They want to see clean class design and efficient usage of priority queues.

## Key Concepts
- **Class Design**: Implementing APIs (insert, delete, get) efficiently.
- **Min-Heap/Max-Heap**: Top K elements, medians, and merging.
- **HashMap Integration**: Combining maps with linked lists or heaps for O(1) access.

## Phase 1: Must-Do (Foundation)

Master these 10 problems to build a solid foundation.

| Problem | Difficulty | Key Concept |
| :--- | :--- | :--- |
| [LRU Cache](https://leetcode.com/problems/lru-cache/) | Medium | HashMap + Doubly Linked List. |
| [Min Stack](https://leetcode.com/problems/min-stack/) | Medium | Two Stacks. |
| [Implement Trie (Prefix Tree)](https://leetcode.com/problems/implement-trie-prefix-tree/) | Medium | Tree with char branches. |
| [Kth Largest Element in an Array](https://leetcode.com/problems/kth-largest-element-in-an-array/) | Medium | Min-Heap or Quick Select. |
| [Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/) | Medium | Min-Heap or Bucket Sort. |
| [Find Median from Data Stream](https://leetcode.com/problems/find-median-from-data-stream/) | Hard | Two Heaps. |
| [Design Tic-Tac-Toe](https://leetcode.com/problems/design-tic-tac-toe/) | Medium | Array/Counter optimization. |
| [Meeting Rooms II](https://leetcode.com/problems/meeting-rooms-ii/) | Medium | Min-Heap (Intervals). |
| [K Closest Points to Origin](https://leetcode.com/problems/k-closest-points-to-origin/) | Medium | Max-Heap. |
| [Design Hit Counter](https://leetcode.com/problems/design-hit-counter/) | Medium | Queue or Fixed Array. |

## Phase 2: Practice & Variants (Depth)

Tackle these 10 harder variations.

| Problem | Difficulty | Key Concept |
| :--- | :--- | :--- |
| [LFU Cache](https://leetcode.com/problems/lfu-cache/) | Hard | HashMap + Freq Map + Linked Lists. |
| [Design Add and Search Words Data Structure](https://leetcode.com/problems/design-add-and-search-words-data-structure/) | Medium | Trie + DFS. |
| [Design In-Memory File System](https://leetcode.com/problems/design-in-memory-file-system/) | Hard | Trie / Directory Tree. |
| [Serialize and Deserialize Binary Tree](https://leetcode.com/problems/serialize-and-deserialize-binary-tree/) | Hard | Tree Design. |
| [Insert Delete GetRandom O(1)](https://leetcode.com/problems/insert-delete-getrandom-o1/) | Medium | HashMap + Array. |
| [Time Based Key-Value Store](https://leetcode.com/problems/time-based-key-value-store/) | Medium | HashMap + Binary Search. |
| [Task Scheduler](https://leetcode.com/problems/task-scheduler/) | Medium | Max-Heap / Greedy. |
| [Reorganize String](https://leetcode.com/problems/reorganize-string/) | Medium | Max-Heap (Greedy). |
| [The Skyline Problem](https://leetcode.com/problems/the-skyline-problem/) | Hard | Heap + Line Sweep. |
| [Maximum Frequency Stack](https://leetcode.com/problems/maximum-frequency-stack/) | Hard | Map of Stacks. |
