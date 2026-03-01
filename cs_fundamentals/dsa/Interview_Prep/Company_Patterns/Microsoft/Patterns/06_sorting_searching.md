# Microsoft Sorting & Searching Patterns

**Frequency**: 🟡 **Medium**

Microsoft frequently asks problems that require custom sorting logic or applying Binary Search in non-obvious ways (e.g., rotated arrays).

## Key Concepts
- **Binary Search**: Not just for sorted arrays, but for "search space" (e.g., finding a minimum capacity).
- **Merge Sort / Quick Sort**: Understanding the divide-and-conquer approach.
- **Custom Comparators**: Sorting objects or strings based on specific rules.
- **Bucket Sort**: For O(N) sorting when ranges are small.

## Phase 1: Must-Do (Foundation)

Master these 10 problems to build a solid foundation.

| Problem | Difficulty | Key Concept |
| :--- | :--- | :--- |
| [Binary Search](https://leetcode.com/problems/binary-search/) | Easy | Standard template. |
| [Search in Rotated Sorted Array](https://leetcode.com/problems/search-in-rotated-sorted-array/) | Medium | Binary Search with condition. |
| [Find Minimum in Rotated Sorted Array](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/) | Medium | Binary Search. |
| [Merge Intervals](https://leetcode.com/problems/merge-intervals/) | Medium | Sorting by start time. |
| [Sort Colors](https://leetcode.com/problems/sort-colors/) | Medium | 3-way partitioning (Dutch National Flag). |
| [Search a 2D Matrix](https://leetcode.com/problems/search-a-2d-matrix/) | Medium | Binary Search on matrix. |
| [Find Peak Element](https://leetcode.com/problems/find-peak-element/) | Medium | Binary Search on unsorted array. |
| [Search Insert Position](https://leetcode.com/problems/search-insert-position/) | Easy | Binary Search lower bound. |
| [Majority Element](https://leetcode.com/problems/majority-element/) | Easy | Sorting or Boyer-Moore. |
| [Valid Anagram](https://leetcode.com/problems/valid-anagram/) | Easy | Sorting or Hash Map. |

## Phase 2: Practice & Variants (Depth)

Tackle these 10 harder variations.

| Problem | Difficulty | Key Concept |
| :--- | :--- | :--- |
| [Median of Two Sorted Arrays](https://leetcode.com/problems/median-of-two-sorted-arrays/) | Hard | Binary Search on partition. |
| [Search a 2D Matrix II](https://leetcode.com/problems/search-a-2d-matrix-ii/) | Medium | Search from top-right. |
| [Find First and Last Position of Element in Sorted Array](https://leetcode.com/problems/find-first-and-last-position-of-element-in-sorted-array/) | Medium | Two Binary Searches. |
| [Wiggle Sort II](https://leetcode.com/problems/wiggle-sort-ii/) | Medium | Quick Select + Virtual Indexing. |
| [Koko Eating Bananas](https://leetcode.com/problems/koko-eating-bananas/) | Medium | Binary Search on Answer. |
| [Capacity To Ship Packages Within D Days](https://leetcode.com/problems/capacity-to-ship-packages-within-d-days/) | Medium | Binary Search on Answer. |
| [Largest Number](https://leetcode.com/problems/largest-number/) | Medium | Custom Comparator. |
| [Sort List](https://leetcode.com/problems/sort-list/) | Medium | Merge Sort on Linked List. |
| [Queue Reconstruction by Height](https://leetcode.com/problems/queue-reconstruction-by-height/) | Medium | Sorting + Insertion. |
| [Count of Smaller Numbers After Self](https://leetcode.com/problems/count-of-smaller-numbers-after-self/) | Hard | Merge Sort / Fenwick Tree. |
