# Adobe Design & Concurrency Patterns

**Frequency**: 🟡 **Medium** (Specific to Adobe)

Unlike many others, Adobe explicitly asks concurrency-related coding questions (threading) and standard system design data structures.

## Key Concepts
- **LRU/LFU**: Standard cache design.
- **Concurrency**: Using Mutex, Semaphores, or language-specific threading primitives (Java `synchronized`, C++ `std::mutex`).
- **Object Oriented Design**: Clean class structure.

## Phase 1: Must-Do (Foundation)

Master these 10 problems to build a solid foundation.

| Problem | Difficulty | Key Concept |
| :--- | :--- | :--- |
| [LRU Cache](https://leetcode.com/problems/lru-cache/) | Medium | Hash Map + Doubly Linked List. |
| [Min Stack](https://leetcode.com/problems/min-stack/) | Medium | Two Stacks. |
| [Implement Queue using Stacks](https://leetcode.com/problems/implement-queue-using-stacks/) | Easy | Stack logic. |
| [Design HashMap](https://leetcode.com/problems/design-hashmap/) | Easy | Array of Lists. |
| [Design HashSet](https://leetcode.com/problems/design-hashset/) | Easy | Array of Lists. |
| [Print in Order](https://leetcode.com/problems/print-in-order/) | Easy | Concurrency (Mutex/Semaphore). |
| [Print FooBar Alternately](https://leetcode.com/problems/print-foobar-alternately/) | Medium | Concurrency. |
| [Design Parking System](https://leetcode.com/problems/design-parking-system/) | Easy | Array/Counter. |
| [Design Underground System](https://leetcode.com/problems/design-underground-system/) | Medium | Hash Maps. |
| [Insert Delete GetRandom O(1)](https://leetcode.com/problems/insert-delete-getrandom-o1/) | Medium | Hash Map + Array. |

## Phase 2: Practice & Variants (Depth)

Tackle these 10 harder variations.

| Problem | Difficulty | Key Concept |
| :--- | :--- | :--- |
| [LFU Cache](https://leetcode.com/problems/lfu-cache/) | Hard | Map + Freq Map + DLL. |
| [Design Twitter](https://leetcode.com/problems/design-twitter/) | Medium | Hash Map + Heap. |
| [Design Browser History](https://leetcode.com/problems/design-browser-history/) | Medium | Two Stacks / DLL. |
| [All O`one Data Structure](https://leetcode.com/problems/all-oone-data-structure/) | Hard | DLL + Map. |
| [Time Based Key-Value Store](https://leetcode.com/problems/time-based-key-value-store/) | Medium | Map + Binary Search. |
| [Building H2O](https://leetcode.com/problems/building-h2o/) | Medium | Concurrency (Barriers). |
| [The Dining Philosophers](https://leetcode.com/problems/the-dining-philosophers/) | Medium | Concurrency (Resource allocation). |
| [Fizz Buzz Multithreaded](https://leetcode.com/problems/fizz-buzz-multithreaded/) | Medium | Concurrency. |
| [Design Hit Counter](https://leetcode.com/problems/design-hit-counter/) | Medium | Queue. |
| [Flatten Nested List Iterator](https://leetcode.com/problems/flatten-nested-list-iterator/) | Medium | Stack / Recursion. |
