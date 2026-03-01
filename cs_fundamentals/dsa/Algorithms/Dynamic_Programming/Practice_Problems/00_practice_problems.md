# Practice Problems: Dynamic Programming

Optimization via subproblems and memoization.

## 1D DP

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Climbing Stairs](https://leetcode.com/problems/climbing-stairs/) | Easy | `dp[i] = dp[i-1] + dp[i-2]`. Fibonacci. |
| [House Robber](https://leetcode.com/problems/house-robber/) | Medium | `dp[i] = max(dp[i-1], dp[i-2] + nums[i])`. |
| [Longest Increasing Subsequence](https://leetcode.com/problems/longest-increasing-subsequence/) | Medium | `dp[i] = max(dp[j]) + 1` for `j < i`. O(n log n) with patience sorting. |
| [Word Break](https://leetcode.com/problems/word-break/) | Medium | `dp[i] = any(dp[j] and s[j:i] in dict)`. |

## 2D/Grid DP

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Unique Paths](https://leetcode.com/problems/unique-paths/) | Medium | `dp[i][j] = dp[i-1][j] + dp[i][j-1]`. |
| [Longest Common Subsequence](https://leetcode.com/problems/longest-common-subsequence/) | Medium | Match: `1 + diag`. No match: `max(left, up)`. |
| [Edit Distance](https://leetcode.com/problems/edit-distance/) | Medium | Insert, Delete, Replace operations. |

## Knapsack

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Coin Change](https://leetcode.com/problems/coin-change/) | Medium | Unbounded knapsack. `min(dp[amount - coin]) + 1`. |
| [Partition Equal Subset Sum](https://leetcode.com/problems/partition-equal-subset-sum/) | Medium | 0/1 Knapsack. Target = sum / 2. |
