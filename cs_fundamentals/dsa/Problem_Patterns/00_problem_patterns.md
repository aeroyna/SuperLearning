# Problem Patterns

Problem patterns are recurring solution templates that appear across different problems. Recognizing these patterns helps quickly identify the right approach and structure solutions efficiently.

## Comprehensive Pattern Index

Many patterns are covered within specific data structure or algorithm sections. Use this index to find them:

| Pattern | Category | Location |
|---------|----------|----------|
| **Sliding Window** | Arrays | [Arrays & Strings](../Data_Structures/Arrays_and_Strings/Sliding_Window/01_sliding_window.md) |
| **Two Pointers** | Arrays | [Arrays & Strings](../Data_Structures/Arrays_and_Strings/Two_Pointers/01_two_pointers.md) |
| **Prefix Sum** | Arrays | [Arrays & Strings](../Data_Structures/Arrays_and_Strings/Prefix_Sum/01_prefix_sum.md) |
| **Fast & Slow Pointers** | Linked Lists | [Linked Lists](../Data_Structures/Linked_Lists/Techniques/01_fast_and_slow_pointers.md) |
| **In-place Reversal** | Linked Lists | [Linked Lists](../Data_Structures/Linked_Lists/Techniques/02_reversing_linked_lists.md) |
| **Monotonic Stack** | Stacks | [Stacks & Queues](../Data_Structures/Stacks_and_Queues/Monotonic_Stack/01_monotonic_stack.md) |
| **Top K Elements** | Heaps | [Heaps](../Data_Structures/Heaps_and_Priority_Queues/Top_K_Problems/01_top_k_problems.md) |
| **Two Heaps** | Heaps | [Heaps](../Data_Structures/Heaps_and_Priority_Queues/02_two_heaps_pattern.md) |
| **Modified Binary Search** | Algorithms | [Binary Search](../Algorithms/Binary_Search/00_binary_search.md) |
| **Backtracking (Subsets)** | Algorithms | [Recursion](../Algorithms/Recursion_and_Backtracking/00_recursion_and_backtracking.md) |
| **Topological Sort** | Graphs | [Advanced Graphs](../Algorithms/Advanced_Graph_Algorithms/Topological_Sort/00_topological_sort.md) |
| **Union Find** | Graphs | [Advanced Graphs](../Algorithms/Advanced_Graph_Algorithms/Minimum_Spanning_Trees/03_union_find.md) |

---

## Specialized Problem Patterns

This section details patterns that often span multiple categories or don't fit neatly into standard data structure buckets.

### Topics

- **[21. Intervals](Intervals/00_intervals.md)** - Overlapping, merging, and scheduling
- **[22. Difference Array](Difference_Array/00_difference_array.md)** - Range update pattern
- **[23. Bit Manipulation](Bit_Manipulation/00_bit_manipulation.md)** - Binary operations and tricks
- **[24. Cyclic Sort](Cyclic_Sort/00_cyclic_sort.md)** - Sorting 1 to n in O(n)
- **[25. Line Sweep](Line_Sweep/00_line_sweep.md)** - Processing events in order (Geometry/Time)
- **[26. K-way Merge](K_Way_Merge/00_k_way_merge.md)** - Merging multiple sorted sources
- **[27. Knapsack](Knapsack/00_knapsack.md)** - 0/1 and Unbounded DP
- **[28. Islands](Islands/00_islands.md)** - Matrix/Grid Traversal
- **[29. Bitmask DP](Bitmask_DP/00_bitmask_dp.md)** - State as integer
- **[30. Quick Select](Quick_Select/00_quick_select.md)** - Kth Element in O(N)

---

## Detailed Problem Lists by Pattern

### 1. Intervals

**Concept**: Dealing with start and end times, finding overlaps, or merging ranges.

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Merge Intervals](https://leetcode.com/problems/merge-intervals/) | Medium | Sort by start time. If `current.start <= last.end`, merge. |
| [Insert Interval](https://leetcode.com/problems/insert-interval/) | Medium | Handle 3 parts: non-overlapping before, overlapping (merge), non-overlapping after. |
| [Non-overlapping Intervals](https://leetcode.com/problems/non-overlapping-intervals/) | Medium | Sort by **end time**. Greedy approach: pick interval ending earliest to leave room for others. |
| [Meeting Rooms](https://leetcode.com/problems/meeting-rooms/) | Easy | Sort start times. Check if any `current.start < prev.end`. |
| [Meeting Rooms II](https://leetcode.com/problems/meeting-rooms-ii/) | Medium | Min-Heap of end times OR Line Sweep (start=+1, end=-1). |
| [Interval List Intersections](https://leetcode.com/problems/interval-list-intersections/) | Medium | Two pointers. Intersection is `[max(start1, start2), min(end1, end2)]`. |
| [Minimum Number of Arrows to Burst Balloons](https://leetcode.com/problems/minimum-number-of-arrows-to-burst-balloons/) | Medium | Same as Non-overlapping Intervals (Sort by end). |

### 2. Difference Array

**Concept**: Efficiently adding a value `k` to a range `[l, r]` in `O(1)`.

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Range Addition](https://leetcode.com/problems/range-addition/) | Medium | `arr[l] += val`, `arr[r+1] -= val`. Prefix sum to get final. |
| [Corporate Flight Bookings](https://leetcode.com/problems/corporate-flight-bookings/) | Medium | Direct application of Difference Array. |
| [Car Pooling](https://leetcode.com/problems/car-pooling/) | Medium | Capacity is the value. `diff[start] += passengers`, `diff[end] -= passengers`. Check if prefix sum > capacity. |

### 3. Bit Manipulation

**Concept**: Using XOR, AND, OR, Shifts to solve problems in `O(1)` space or faster.

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Single Number](https://leetcode.com/problems/single-number/) | Easy | `a ^ a = 0`. XOR all numbers. |
| [Counting Bits](https://leetcode.com/problems/counting-bits/) | Easy | DP: `count[i] = count[i >> 1] + (i & 1)`. |
| [Reverse Bits](https://leetcode.com/problems/reverse-bits/) | Easy | Iterate 32 times, shift result left, add last bit of n, shift n right. |
| [Sum of Two Integers](https://leetcode.com/problems/sum-of-two-integers/) | Medium | Adder logic: `(a ^ b)` is sum without carry, `(a & b) << 1` is carry. |
| [Subsets](https://leetcode.com/problems/subsets/) | Medium | Iterate `0` to `2^n - 1`. Bitmask determines inclusion. |

### 4. Cyclic Sort

**Concept**: Sorting numbers `1` to `n` in `O(n)` time by placing `val` at index `val - 1`.

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Missing Number](https://leetcode.com/problems/missing-number/) | Easy | Cyclic sort `0` to `n`. Index where `nums[i] != i` is missing. |
| [Find the Duplicate Number](https://leetcode.com/problems/find-the-duplicate-number/) | Medium | Cyclic sort. If `nums[i] != i+1` and `nums[nums[i]-1] == nums[i]`, it's duplicate. |
| [First Missing Positive](https://leetcode.com/problems/first-missing-positive/) | Hard | Ignore negatives and `> n`. Place `nums[i]` at `nums[i]-1`. First index mismatch is answer. |

### 5. Line Sweep

**Concept**: Visualize a line sweeping across the plane. Process "events" (start/end) in sorted order.

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [The Skyline Problem](https://leetcode.com/problems/the-skyline-problem/) | Hard | Events: `(L, -H)`, `(R, H)`. Max-Heap stores active heights. |
| [My Calendar III](https://leetcode.com/problems/my-calendar-iii/) | Hard | `(start, +1)`, `(end, -1)`. Max prefix sum of events (using TreeMap). |
| [Rectangle Area II](https://leetcode.com/problems/rectangle-area-ii/) | Hard | Sweep line on X-axis. Maintain active Y-intervals (Segment Tree helps). |

### 6. K-way Merge

**Concept**: Efficiently merging K sorted streams using a Heap.

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Merge K Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/) | Hard | Min-Heap of size K. Push heads, pop min, push next. |
| [Kth Smallest Number in M Sorted Lists](https://leetcode.com/problems/kth-smallest-element-in-a-sorted-matrix/) | Medium | Same as merge. Stop after Kth pop. |
| [Smallest Range Covering Elements from K Lists](https://leetcode.com/problems/smallest-range-covering-elements-from-k-lists/) | Hard | Min-Heap of size K. Track current max. Range = `curMax - heap.min`. |
| [Find K Pairs with Smallest Sums](https://leetcode.com/problems/find-k-pairs-with-smallest-sums/) | Medium | Merge K streams implicitly defined by `nums1[i] + nums2[j]`. |

### 7. Knapsack

**Concept**: Select items with weight/value to max value capacity. 0/1 or Unbounded.

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Partition Equal Subset Sum](https://leetcode.com/problems/partition-equal-subset-sum/) | Medium | 0/1 Knapsack. Capacity = Sum/2. |
| [Target Sum](https://leetcode.com/problems/target-sum/) | Medium | 0/1 Knapsack. Find subset P with `sum(P) = (S + total)/2`. |
| [Coin Change](https://leetcode.com/problems/coin-change/) | Medium | Unbounded Knapsack (Min). `dp[w] = min(dp[w], 1 + dp[w-coin])`. |
| [Coin Change II](https://leetcode.com/problems/coin-change-ii/) | Medium | Unbounded Knapsack (Ways). `dp[w] += dp[w-coin]`. |

### 8. Islands (Matrix Traversal)

**Concept**: DFS/BFS on a grid to find connected components or paths.

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Number of Islands](https://leetcode.com/problems/number-of-islands/) | Medium | DFS/BFS to mark connected '1's as visited. |
| [Max Area of Island](https://leetcode.com/problems/max-area-of-island/) | Medium | DFS returns count of nodes in component. |
| [Rotting Oranges](https://leetcode.com/problems/rotting-oranges/) | Medium | Multi-source BFS from all rotten oranges. |
| [Word Search](https://leetcode.com/problems/word-search/) | Medium | DFS + Backtracking. |

### 9. Bitmask DP

**Concept**: DP on subsets using bitmasks (N <= 20).

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Smallest Sufficient Team](https://leetcode.com/problems/smallest-sufficient-team/) | Hard | `dp[mask]` = min size team to cover skill mask. |
| [Partition to K Equal Sum Subsets](https://leetcode.com/problems/partition-to-k-equal-sum-subsets/) | Medium | `dp[mask]` = sum of current subset. |
| [Shortest Path Visiting All Nodes](https://leetcode.com/problems/shortest-path-visiting-all-nodes/) | Hard | BFS state: `(node, mask)`. |

### 10. Quick Select

**Concept**: Partitioning to find Kth element.

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Kth Largest Element in an Array](https://leetcode.com/problems/kth-largest-element-in-an-array/) | Medium | Partition array. If pivot index == k, return value. |
| [K Closest Points to Origin](https://leetcode.com/problems/k-closest-points-to-origin/) | Medium | Quick Select on distance squared. |

---

## Pattern Recognition Guide

### How to identify which pattern to use?

| Signal | Potential Pattern |
|--------|-------------------|
| "Contiguous subarray", "substring" | **Sliding Window** |
| "Sorted array", "target sum", "triplet" | **Two Pointers** |
| "Range sum", "subarray sum equals k" | **Prefix Sum** |
| "Cycle in list", "middle of list" | **Fast & Slow Pointers** |
| "Next greater element", "histogram area" | **Monotonic Stack** |
| "Top k items", "merge k sorted" | **Top K Elements (Heap)** |
| "Overlapping time slots", "merge ranges" | **Intervals** |
| "Add to range [l, r]", "flight bookings" | **Difference Array** |
| "Array 1 to n", "missing positive" | **Cyclic Sort** |
| "Max overlapping intervals", "skyline" | **Line Sweep** |
| "Merge K sorted", "Kth in M arrays" | **K-way Merge** |
| "XOR", "single number", "subsets" | **Bit Manipulation** |
| "Knapsack", "subset sum", "coin change" | **Knapsack (DP)** |
| "Grid", "islands", "connected components" | **Islands (DFS/BFS)** |
| "N <= 20", "Assign N to N" | **Bitmask DP** |
| "Kth smallest/largest", "O(N)" | **Quick Select** |