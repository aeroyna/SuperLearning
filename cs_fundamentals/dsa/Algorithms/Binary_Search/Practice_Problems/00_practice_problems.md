# Practice Problems: Binary Search

Searching in O(log n) on sorted or monotonic data.

## Standard & Rotated

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Binary Search](https://leetcode.com/problems/binary-search/) | Easy | Standard template. |
| [Search in Rotated Sorted Array](https://leetcode.com/problems/search-in-rotated-sorted-array/) | Medium | Determine which half is sorted, then check if target is in range. |
| [Find Minimum in Rotated Sorted Array](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/) | Medium | Compare `mid` with `right` to find inflection point. |

## Answer Space

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Koko Eating Bananas](https://leetcode.com/problems/koko-eating-bananas/) | Medium | Binary search on speed [1, max(piles)]. Check feasibility. |
| [Median of Two Sorted Arrays](https://leetcode.com/problems/median-of-two-sorted-arrays/) | Hard | Partitioning two arrays to ensure left halves have equal size/elements. |
| [Search Insert Position](https://leetcode.com/problems/search-insert-position/) | Easy | Lower bound pattern. |
