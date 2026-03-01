# Adobe Interview Patterns (2024-2025)

Adobe's interviews are a balanced mix of algorithmic problem-solving, system design, and, notably, concurrency/multithreading. They value strong fundamentals in core data structures and practical coding skills.

## 📊 2024-2025 Trends & Shift

*   **Balanced Fundamentals**: A strong mix of Arrays, Strings, Linked Lists, Trees, and DP. No single topic dominates overwhelmingly, requiring broad competence.
*   **Concurrency Emphasis**: Unlike many peers, Adobe frequently asks specific multithreading questions (e.g., `Print in Order`, `Task Scheduler`).
*   **Practical Coding**: Questions often map to real-world scenarios like text editors, cache management (`LRU`), or file systems.
*   **System Design**: Even for SDE-2 roles, expect dedicated rounds or questions on designing scalable systems (e.g., "Design a Rate Limiter").

---

## 🏆 Top Patterns by Frequency

Based on recent interview reports, these are the highest-yield patterns to master for Adobe. Click on a pattern to see the detailed problem list.

### 1. [Arrays & Strings](Patterns/01_arrays_strings.md)
**Frequency**: 🔴 **Very High**
Focus: Two pointers, sliding window, parsing, math simulation.

### 2. [Trees & Graphs](Patterns/02_trees_graphs.md)
**Frequency**: 🟠 **High**
Focus: Traversals, BST properties, LCA, connected components.

### 3. [Dynamic Programming](Patterns/03_dynamic_programming.md)
**Frequency**: 🟠 **High**
Focus: 1D optimization, grid paths, knapsack variants, string editing.

### 4. [Linked Lists](Patterns/04_linked_lists.md)
**Frequency**: 🟡 **Medium**
Focus: Reversal, merging, cycle detection, manipulation.

### 5. [Design & Concurrency](Patterns/05_design_concurrency.md)
**Frequency**: 🟡 **Medium (Adobe Specific)**
Focus: Threading primitives, cache design, class structure.

### 6. [Sorting & Math](Patterns/06_sorting_math.md)
**Frequency**: 🟡 **Medium**
Focus: Binary search, custom sorts, overflow handling, number theory.

---

## 🎯 The "Must-Do" List (Top 10)

If you are short on time, master these 10 classic Adobe questions.

1.  **[Two Sum](https://leetcode.com/problems/two-sum/)** (Hash Map)
2.  **[Add Two Numbers](https://leetcode.com/problems/add-two-numbers/)** (Linked List)
3.  **[Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/)** (Sliding Window)
4.  **[Median of Two Sorted Arrays](https://leetcode.com/problems/median-of-two-sorted-arrays/)** (Binary Search)
5.  **[LRU Cache](https://leetcode.com/problems/lru-cache/)** (Design)
6.  **[Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/)** (Pointers)
7.  **[String to Integer (atoi)](https://leetcode.com/problems/string-to-integer-atoi/)** (Parsing)
8.  **[Merge k Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/)** (Heap)
9.  **[Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/)** (Two Pointers)
10. **[Climbing Stairs](https://leetcode.com/problems/climbing-stairs/)** (DP)

---

## 📅 1-Month Study Plan

Follow this structured plan to cover 10 problems/day across all key patterns.
[**View the 30-Day Adobe Plan**](adobe_1_month_plan.md)

---

## 🧠 Adobe-Specific Tips

1.  **Brush up on Threads**: Review `Mutex`, `Semaphore`, `synchronized` (Java), or `std::thread` (C++). You *will* likely see a concurrency question.
2.  **Think "Product"**: For design questions, think about how a user interacts with the system (e.g., "What happens if the cache is full? What is the eviction policy?").
3.  **Clean Code**: Adobe engineers value readability and modularity. Avoid "hacky" one-liners in favor of clear, maintainable functions.
4.  **Java/C++ Knowledge**: If you claim expertise in a language, expect deep-dive questions about its internals (Memory model, Garbage collection, v-tables).

## Related Learning Paths
- **[Array & String Patterns](../../../Data_Structures/Arrays_and_Strings/00_arrays_and_strings.md)**
- **[Dynamic Programming Patterns](../../../Problem_Patterns/Knapsack/00_knapsack.md)**
- **[System Design Patterns](../../../Problem_Patterns/00_problem_patterns.md)**
