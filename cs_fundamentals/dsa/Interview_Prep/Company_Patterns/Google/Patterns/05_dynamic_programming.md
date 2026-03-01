# Google Dynamic Programming Patterns

**Frequency**: 🟡 **Medium**

While "pure" math DP is less common now, Google still asks DP problems that involve grids, strings, or optimizing recursive solutions (Memoization).

## Key Concepts
- **Memoization**: Caching results of expensive function calls.
- **Grid Paths**: Counting paths or finding min/max path sums.
- **String Alignment**: Edit distance, wildcard matching.
- **Bitmask DP**: Handling state for small N (e.g., TSP).

## Phase 1: Must-Do (Foundation)

Master these 10 problems to build a solid foundation.

| Problem | Difficulty | Key Concept |
| :--- | :--- | :--- |
| [Climbing Stairs](https://leetcode.com/problems/climbing-stairs/) | Easy | Basic Fibonacci DP. |
| [Word Break](https://leetcode.com/problems/word-break/) | Medium | 1D DP. |
| [Longest Increasing Subsequence](https://leetcode.com/problems/longest-increasing-subsequence/) | Medium | 1D DP (O(N^2) or O(N log N)). |
| [Coin Change](https://leetcode.com/problems/coin-change/) | Medium | Unbounded Knapsack. |
| [Unique Paths](https://leetcode.com/problems/unique-paths/) | Medium | Grid DP. |
| [Minimum Path Sum](https://leetcode.com/problems/minimum-path-sum/) | Medium | Grid DP. |
| [Longest Common Subsequence](https://leetcode.com/problems/longest-common-subsequence/) | Medium | String DP. |
| [Maximum Product Subarray](https://leetcode.com/problems/maximum-product-subarray/) | Medium | Tracking min and max product. |
| [Best Time to Buy and Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/) | Easy | One pass tracking min price. |
| [House Robber](https://leetcode.com/problems/house-robber/) | Medium | 1D DP. |

## Phase 2: Practice & Variants (Depth)

Tackle these 10 harder variations.

| Problem | Difficulty | Key Concept |
| :--- | :--- | :--- |
| [Edit Distance](https://leetcode.com/problems/edit-distance/) | Medium | 2D DP (String alignment). |
| [Regular Expression Matching](https://leetcode.com/problems/regular-expression-matching/) | Hard | 2D DP / Recursion. |
| [Longest Palindromic Substring](https://leetcode.com/problems/longest-palindromic-substring/) | Medium | Center expansion or DP. |
| [Wildcard Matching](https://leetcode.com/problems/wildcard-matching/) | Hard | 2D DP. |
| [Split Array Largest Sum](https://leetcode.com/problems/split-array-largest-sum/) | Hard | DP or Binary Search on Answer. |
| [Burst Balloons](https://leetcode.com/problems/burst-balloons/) | Hard | Interval DP. |
| [Student Attendance Record II](https://leetcode.com/problems/student-attendance-record-ii/) | Hard | DP with state (A count, L streak). |
| [Palindromic Substrings](https://leetcode.com/problems/palindromic-substrings/) | Medium | Center expansion. |
| [Partition Equal Subset Sum](https://leetcode.com/problems/partition-equal-subset-sum/) | Medium | 0/1 Knapsack. |
| [Maximum Profit in Job Scheduling](https://leetcode.com/problems/maximum-profit-in-job-scheduling/) | Hard | DP + Binary Search. |