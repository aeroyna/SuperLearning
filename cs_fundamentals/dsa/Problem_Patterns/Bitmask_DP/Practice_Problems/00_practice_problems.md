# Practice Problems: Bitmask DP

Optimizing state representation with bits.

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Smallest Sufficient Team](https://leetcode.com/problems/smallest-sufficient-team/) | Hard | `dp[mask]` = min size team to cover skill mask. |
| [Partition to K Equal Sum Subsets](https://leetcode.com/problems/partition-to-k-equal-sum-subsets/) | Medium | `dp[mask]` = sum of current subset. If sum == target, reset to 0. |
| [Shortest Path Visiting All Nodes](https://leetcode.com/problems/shortest-path-visiting-all-nodes/) | Hard | BFS state: `(node, mask)`. |
| [Maximum Students Taking Exam](https://leetcode.com/problems/maximum-students-taking-exam/) | Hard | `dp[row][mask]` depends on `dp[row-1][prev_mask]`. Check compatibility. |
| [Beautiful Arrangement](https://leetcode.com/problems/beautiful-arrangement/) | Medium | `dp[mask]` = ways to arrange used numbers. |
