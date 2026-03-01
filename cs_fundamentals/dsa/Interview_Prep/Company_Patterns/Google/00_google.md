# Google Interview Patterns (2024-2025)

Google interviews are renowned for their emphasis on algorithmic intuition, graph theory, and the ability to handle ambiguity. Recent data from 2024-2025 indicates a shift towards more graph-heavy and design-oriented coding problems.

## 📊 2024-2025 Trends & Shift

*   **Graph Surge**: A significant increase in graph problems, especially BFS/DFS on grids and Union-Find.
*   **Design Hybrids**: Problems that require defining a class with specific methods (e.g., `SnapshotArray`, `LRUCache`) are very common.
*   **"Hidden" DP**: Pure mathematical DP is less common; instead, look for DP on grids, strings, or memoization within DFS.
*   **Intervals**: Scheduling and merging intervals remain a staple.

---

## 🏆 Top Patterns by Frequency

Based on recent interview reports, these are the highest-yield patterns to master for Google. Click on a pattern to see the detailed problem list.

### 1. [Graphs (BFS / DFS / Union-Find)](Patterns/01_graphs.md)
**Frequency**: 🔴 **Very High**
Focus: Grid-based problems, connected components, topological sort.

### 2. [Intervals & Sweeping](Patterns/02_intervals_sweeping.md)
**Frequency**: 🟠 **High**
Focus: Scheduling, merging, line sweep algorithms.

### 3. [Arrays & Strings](Patterns/03_arrays_strings.md)
**Frequency**: 🟠 **High**
Focus: Sliding window, two pointers, prefix sums.

### 4. [System Design Hybrids](Patterns/04_design_hybrids.md)
**Frequency**: 📈 **Increasing**
Focus: Class design, API implementation, combining data structures (e.g., Hash Map + Linked List).

### 5. [Dynamic Programming](Patterns/05_dynamic_programming.md)
**Frequency**: 🟡 **Medium**
Focus: Grid paths, string optimization, memoization.

### 6. [Trees](Patterns/06_trees.md)
**Frequency**: 🟡 **Medium**
Focus: Serialization, LCA, path finding.

### 7. [Heaps & Priority Queues](Patterns/07_heaps_pq.md)
**Frequency**: 🟡 **Medium**
Focus: Top K, merging sorted collections, median finding.

### 8. [Advanced & Niche Patterns](Patterns/08_advanced_niche.md)
**Frequency**: ⚪ **Low (but High Signal)**
Focus: Segment Trees, Fenwick Trees, Rolling Hash, specialized search.

---

## 🎯 The "Must-Do" List (Top 10)

If you are short on time, master these 10 problems. They cover the widest range of Google's favorite patterns.

1.  **[Number of Islands](https://leetcode.com/problems/number-of-islands/)** (Graph BFS/DFS)
2.  **[Meeting Rooms II](https://leetcode.com/problems/meeting-rooms-ii/)** (Heap/Intervals)
3.  **[Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/)** (Two Pointers)
4.  **[LRU Cache](https://leetcode.com/problems/lru-cache/)** (Design)
5.  **[Merge K Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/)** (Heap)
6.  **[Word Break](https://leetcode.com/problems/word-break/)** (DP/Trie)
7.  **[Course Schedule II](https://leetcode.com/problems/course-schedule-ii/)** (Topological Sort)
8.  **[Decode String](https://leetcode.com/problems/decode-string/)** (Stack/Recursion)
9.  **[Longest Increasing Path in a Matrix](https://leetcode.com/problems/longest-increasing-path-in-a-matrix/)** (DFS + Memo)
10. **[Snapshot Array](https://leetcode.com/problems/snapshot-array/)** (Binary Search / Design)

---

## 📅 1-Month Study Plan

Follow this structured plan to cover 10 problems/day across all key patterns.
[**View the 30-Day Google Plan**](google_1_month_plan.md)

---

## 🧠 Google-Specific Tips

1.  **Clarify Ambiguity**: Google questions are often intentionally vague. Ask about constraints, input format, and edge cases *before* coding.
2.  **Think "Scale"**: Always ask yourself, "What if the input is too large to fit in memory?" (Streaming algorithms, external sort).
3.  **Code Quality**: Write clean, modular code. Use helper functions. Google interviewers care about readability as much as correctness.
4.  **Test Your Code**: Dry run your code with a sample case manually before saying "I'm done."

## Related Learning Paths
- **[Graph Patterns](../../../Problem_Patterns/Islands/00_islands.md)**
- **[Interval Patterns](../../../Problem_Patterns/Intervals/00_intervals.md)**
- **[Bitmask DP](../../../Problem_Patterns/Bitmask_DP/00_bitmask_dp.md)**
