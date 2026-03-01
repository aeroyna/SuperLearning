# Dynamic Programming

Dynamic programming (DP) is an optimization technique that solves problems by breaking them into overlapping subproblems and storing results to avoid redundant computation.

## Overview

DP = Recursion + Memoization (or Bottom-up Tabulation)

## Topics

- [16.1 DP Framework](01_dp_framework.md)
- [16.2 1D DP Problems](One_Dimensional/01_1d_dp_problems.md)
- [16.3 2D DP Problems](Multi_Dimensional/01_2d_dp_problems.md)
- [16.4 Matrix DP](Multi_Dimensional/02_matrix_dp.md)
- [16.5 DP on Strings](Common_Patterns/01_dp_on_strings.md)
- [16.6 State Machine DP](Common_Patterns/02_state_machine_dp.md)
- [16.7 Practice Problems](Practice_Problems/00_practice_problems.md)

## When to Use DP

1. **Optimal value** (max/min) or **count of ways**
2. **Overlapping subproblems** - Same subproblems solved multiple times
3. **Optimal substructure** - Optimal solution contains optimal sub-solutions
4. **Decisions affect future** - Unlike greedy

## Framework

### Step 1: Define State

What information describes a subproblem?
- Index position
- Remaining capacity
- Boolean flags

### Step 2: Define Recurrence

How does dp[i] relate to previous states?

### Step 3: Base Cases

What are the smallest subproblems?

### Step 4: Order of Computation

Ensure dependencies are computed first.

## Top-Down (Memoization)

>[!example]- C++
>```cpp
>unordered_map<int, int> memo;
>
>int fibonacci(int n) {
>    if (memo.count(n)) return memo[n];
>    if (n <= 1) return n;
>
>    memo[n] = fibonacci(n - 1) + fibonacci(n - 2);
>    return memo[n];
>}
>```

>[!example]- Java
>```java
>Map<Integer, Integer> memo = new HashMap<>();
>
>public int fibonacci(int n) {
>    if (memo.containsKey(n)) return memo.get(n);
>    if (n <= 1) return n;
>
>    int result = fibonacci(n - 1) + fibonacci(n - 2);
>    memo.put(n, result);
>    return result;
>}
>```

>[!example]- Python
>```python
>def fibonacci(n, memo={}):
>    if n in memo:
>        return memo[n]
>    if n <= 1:
>        return n
>
>    memo[n] = fibonacci(n-1) + fibonacci(n-2)
>    return memo[n]
>```

>[!example]- JavaScript
>```javascript
>const memo = new Map();
>
>function fibonacci(n) {
>    if (memo.has(n)) return memo.get(n);
>    if (n <= 1) return n;
>
>    const result = fibonacci(n - 1) + fibonacci(n - 2);
>    memo.set(n, result);
>    return result;
>}
>```

## Bottom-Up (Tabulation)

>[!example]- C++
>```cpp
>int fibonacci(int n) {
>    if (n <= 1) return n;
>
>    vector<int> dp(n + 1);
>    dp[1] = 1;
>
>    for (int i = 2; i <= n; i++) {
>        dp[i] = dp[i - 1] + dp[i - 2];
>    }
>
>    return dp[n];
>}
>```

>[!example]- Java
>```java
>public int fibonacci(int n) {
>    if (n <= 1) return n;
>
>    int[] dp = new int[n + 1];
>    dp[1] = 1;
>
>    for (int i = 2; i <= n; i++) {
>        dp[i] = dp[i - 1] + dp[i - 2];
>    }
>
>    return dp[n];
>}
>```

>[!example]- Python
>```python
>def fibonacci(n):
>    if n <= 1:
>        return n
>
>    dp = [0] * (n + 1)
>    dp[1] = 1
>
>    for i in range(2, n + 1):
>        dp[i] = dp[i-1] + dp[i-2]
>
>    return dp[n]
>```

>[!example]- JavaScript
>```javascript
>function fibonacci(n) {
>    if (n <= 1) return n;
>
>    const dp = new Array(n + 1).fill(0);
>    dp[1] = 1;
>
>    for (let i = 2; i <= n; i++) {
>        dp[i] = dp[i - 1] + dp[i - 2];
>    }
>
>    return dp[n];
>}
>```

## Classic DP Problems

### Climbing Stairs

```python
def climbStairs(n):
    if n <= 2:
        return n
    prev2, prev1 = 1, 2
    for _ in range(3, n + 1):
        prev2, prev1 = prev1, prev2 + prev1
    return prev1
```

### House Robber

```python
def rob(nums):
    if not nums:
        return 0
    if len(nums) == 1:
        return nums[0]

    prev2, prev1 = 0, nums[0]
    for i in range(1, len(nums)):
        prev2, prev1 = prev1, max(prev1, prev2 + nums[i])
    return prev1
```

### Longest Increasing Subsequence

```python
def lengthOfLIS(nums):
    if not nums:
        return 0

    dp = [1] * len(nums)

    for i in range(1, len(nums)):
        for j in range(i):
            if nums[j] < nums[i]:
                dp[i] = max(dp[i], dp[j] + 1)

    return max(dp)
```

### Longest Common Subsequence

```python
def longestCommonSubsequence(text1, text2):
    m, n = len(text1), len(text2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]

    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if text1[i-1] == text2[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])

    return dp[m][n]
```

## DP Categories

| Category | Examples |
|----------|----------|
| 1D Linear | Climbing Stairs, House Robber |
| 2D String | LCS, Edit Distance |
| Matrix Path | Unique Paths, Min Path Sum |
| Knapsack | 0/1 Knapsack, Coin Change |
| Interval | Burst Balloons, Matrix Chain |
| State Machine | Best Time to Buy Stock |

## Complexity Analysis

- **Time**: O(states × work per state)
- **Space**: O(states), often reducible

## Key Interview Problems

| Problem | Type | Difficulty | LeetCode Link |
| --------- | ------ | ------------ | --- |
| Climbing Stairs | 1D | Easy | [Link](https://leetcode.com/problems/climbing-stairs/) |
| House Robber | 1D | Medium | [Link](https://leetcode.com/problems/house-robber/) |
| Coin Change | Knapsack | Medium | [Link](https://leetcode.com/problems/coin-change/) |
| Longest Increasing Subsequence | 1D | Medium | [Link](https://leetcode.com/problems/longest-increasing-subsequence/) |
| Longest Common Subsequence | 2D String | Medium | [Link](https://leetcode.com/problems/longest-common-subsequence/) |
| Edit Distance | 2D String | Medium | [Link](https://leetcode.com/problems/edit-distance/) |
| Unique Paths | Matrix | Medium | [Link](https://leetcode.com/problems/unique-paths/) |
| Word Break | 1D | Medium | [Link](https://leetcode.com/problems/word-break/) |
