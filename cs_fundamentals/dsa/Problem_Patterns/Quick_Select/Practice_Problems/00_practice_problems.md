# Practice Problems: Quick Select

Finding Kth element efficiently.

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Kth Largest Element in an Array](https://leetcode.com/problems/kth-largest-element-in-an-array/) | Medium | Partition array. If pivot index == k, return value. |
| [K Closest Points to Origin](https://leetcode.com/problems/k-closest-points-to-origin/) | Medium | Quick Select on distance squared. |
| [Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/) | Medium | Quick Select on frequency counts (bucket sort also works). |
| [Wiggle Sort II](https://leetcode.com/problems/wiggle-sort-ii/) | Medium | Find median via Quick Select. Map indices to place `< med`, `> med`, `= med`. |
