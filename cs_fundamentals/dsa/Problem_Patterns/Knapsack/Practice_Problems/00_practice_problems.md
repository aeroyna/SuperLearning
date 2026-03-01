# Practice Problems: Knapsack

Optimization problems using DP.

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Partition Equal Subset Sum](https://leetcode.com/problems/partition-equal-subset-sum/) | Medium | 0/1 Knapsack. Capacity = Sum/2. |
| [Target Sum](https://leetcode.com/problems/target-sum/) | Medium | 0/1 Knapsack. Find subset P with `sum(P) = (S + total)/2`. |
| [Coin Change](https://leetcode.com/problems/coin-change/) | Medium | Unbounded Knapsack (Min). `dp[w] = min(dp[w], 1 + dp[w-coin])`. |
| [Coin Change II](https://leetcode.com/problems/coin-change-ii/) | Medium | Unbounded Knapsack (Ways). `dp[w] += dp[w-coin]`. |
| [Ones and Zeroes](https://leetcode.com/problems/ones-and-zeroes/) | Medium | 2D Knapsack. Costs are 0s count and 1s count. |
