# Multi-Dimensional Dynamic Programming

Multi-dimensional DP uses two or more state variables to represent subproblems. Common forms include 2D DP for string problems, matrix DP for grid traversal, and state space DP for complex constraints.

## Overview

In multi-dimensional DP:
- State: `dp[i][j]` (or more dimensions)
- Each dimension represents a different aspect of the subproblem
- Transition involves combinations of previous states

## Topics

- [16.3.1 2D DP Problems](01_2d_dp_problems.md)
- [16.4.1 Matrix DP](02_matrix_dp.md)

## Common 2D State Definitions

| Type | State Meaning | Example |
|------|---------------|---------|
| Two pointers | dp[i][j] = answer for s1[0..i] and s2[0..j] | LCS |
| Grid | dp[i][j] = answer reaching cell (i, j) | Unique Paths |
| Interval | dp[i][j] = answer for substring [i, j] | Palindrome |
| Knapsack | dp[i][j] = answer using i items with capacity j | 0/1 Knapsack |

## String DP Problems

### Longest Common Subsequence

>[!example]- C++
>```cpp
>int longestCommonSubsequence(string text1, string text2) {
>    int m = text1.length(), n = text2.length();
>    vector<vector<int>> dp(m + 1, vector<int>(n + 1, 0));
>    
>    for (int i = 1; i <= m; i++) {
>        for (int j = 1; j <= n; j++) {
>            if (text1[i - 1] == text2[j - 1]) {
>                dp[i][j] = dp[i - 1][j - 1] + 1;
>            } else {
>                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
>            }
>        }
>    }
>    return dp[m][n];
>}
>```

>[!example]- Java
>```java
>public int longestCommonSubsequence(String text1, String text2) {
>    int m = text1.length(), n = text2.length();
>    int[][] dp = new int[m + 1][n + 1];
>    
>    for (int i = 1; i <= m; i++) {
>        for (int j = 1; j <= n; j++) {
>            if (text1.charAt(i - 1) == text2.charAt(j - 1)) {
>                dp[i][j] = dp[i - 1][j - 1] + 1;
>            } else {
>                dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
>            }
>        }
>    }
>    return dp[m][n];
>}
>```

>[!example]- Python
>```python
>def lcs(text1, text2):
>    m, n = len(text1), len(text2)
>    dp = [[0] * (n + 1) for _ in range(m + 1)]
>
>    for i in range(1, m + 1):
>        for j in range(1, n + 1):
>            if text1[i-1] == text2[j-1]:
>                dp[i][j] = dp[i-1][j-1] + 1
>            else:
>                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
>
>    return dp[m][n]
>```

>[!example]- JavaScript
>```javascript
>function longestCommonSubsequence(text1, text2) {
>    const m = text1.length, n = text2.length;
>    const dp = Array.from({ length: m + 1 }, () => Array(n + 1).fill(0));
>    
>    for (let i = 1; i <= m; i++) {
>        for (let j = 1; j <= n; j++) {
>            if (text1[i - 1] === text2[j - 1]) {
>                dp[i][j] = dp[i - 1][j - 1] + 1;
>            } else {
>                dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
>            }
>        }
>    }
>    return dp[m][n];
>}
>```

**State**: `dp[i][j]` = LCS length of text1[0..i-1] and text2[0..j-1]
**Recurrence**:
- Match: `dp[i][j] = dp[i-1][j-1] + 1`
- No match: `dp[i][j] = max(dp[i-1][j], dp[i][j-1])`

### Edit Distance

```python
def min_distance(word1, word2):
    m, n = len(word1), len(word2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]

    # Base cases
    for i in range(m + 1):
        dp[i][0] = i  # Delete all
    for j in range(n + 1):
        dp[0][j] = j  # Insert all

    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if word1[i-1] == word2[j-1]:
                dp[i][j] = dp[i-1][j-1]  # No operation needed
            else:
                dp[i][j] = 1 + min(
                    dp[i-1][j],      # Delete
                    dp[i][j-1],      # Insert
                    dp[i-1][j-1]     # Replace
                )

    return dp[m][n]
```

**State**: `dp[i][j]` = min operations to convert word1[0..i-1] to word2[0..j-1]

## Matrix DP Problems

### Unique Paths

>[!example]- C++
>```cpp
>int uniquePaths(int m, int n) {
>    vector<vector<int>> dp(m, vector<int>(n, 1));
>    
>    for (int i = 1; i < m; i++) {
>        for (int j = 1; j < n; j++) {
>            dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
>        }
>    }
>    return dp[m - 1][n - 1];
>}
>```

>[!example]- Java
>```java
>public int uniquePaths(int m, int n) {
>    int[][] dp = new int[m][n];
>    for (int[] row : dp) Arrays.fill(row, 1);
>    
>    for (int i = 1; i < m; i++) {
>        for (int j = 1; j < n; j++) {
>            dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
>        }
>    }
>    return dp[m - 1][n - 1];
>}
>```

>[!example]- Python
>```python
>def unique_paths(m, n):
>    dp = [[1] * n for _ in range(m)]
>
>    for i in range(1, m):
>        for j in range(1, n):
>            dp[i][j] = dp[i-1][j] + dp[i][j-1]
>
>    return dp[m-1][n-1]
>```

>[!example]- JavaScript
>```javascript
>function uniquePaths(m, n) {
>    const dp = Array.from({ length: m }, () => Array(n).fill(1));
>    
>    for (let i = 1; i < m; i++) {
>        for (let j = 1; j < n; j++) {
>            dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
>        }
>    }
>    return dp[m - 1][n - 1];
>}
>```

**State**: `dp[i][j]` = paths from (0,0) to (i,j)
**Recurrence**: `dp[i][j] = dp[i-1][j] + dp[i][j-1]` (from above or left)

### Minimum Path Sum

>[!example]- C++
>```cpp
>int minPathSum(vector<vector<int>>& grid) {
>    int m = grid.size(), n = grid[0].size();
>    vector<vector<int>> dp(m, vector<int>(n, 0));
>    
>    dp[0][0] = grid[0][0];
>    for (int i = 1; i < m; i++) dp[i][0] = dp[i - 1][0] + grid[i][0];
>    for (int j = 1; j < n; j++) dp[0][j] = dp[0][j - 1] + grid[0][j];
>    
>    for (int i = 1; i < m; i++) {
>        for (int j = 1; j < n; j++) {
>            dp[i][j] = min(dp[i - 1][j], dp[i][j - 1]) + grid[i][j];
>        }
>    }
>    return dp[m - 1][n - 1];
>}
>```

>[!example]- Java
>```java
>public int minPathSum(int[][] grid) {
>    int m = grid.length, n = grid[0].length;
>    int[][] dp = new int[m][n];
>    
>    dp[0][0] = grid[0][0];
>    for (int i = 1; i < m; i++) dp[i][0] = dp[i - 1][0] + grid[i][0];
>    for (int j = 1; j < n; j++) dp[0][j] = dp[0][j - 1] + grid[0][j];
>    
>    for (int i = 1; i < m; i++) {
>        for (int j = 1; j < n; j++) {
>            dp[i][j] = Math.min(dp[i - 1][j], dp[i][j - 1]) + grid[i][j];
>        }
>    }
>    return dp[m - 1][n - 1];
>}
>```

>[!example]- Python
>```python
>def min_path_sum(grid):
>    m, n = len(grid), len(grid[0])
>    dp = [[0] * n for _ in range(m)]
>
>    dp[0][0] = grid[0][0]
>
>    # First row
>    for j in range(1, n):
>        dp[0][j] = dp[0][j-1] + grid[0][j]
>
>    # First column
>    for i in range(1, m):
>        dp[i][0] = dp[i-1][0] + grid[i][0]
>
>    for i in range(1, m):
>        for j in range(1, n):
>            dp[i][j] = grid[i][j] + min(dp[i-1][j], dp[i][j-1])
>
>    return dp[m-1][n-1]
>```

>[!example]- JavaScript
>```javascript
>function minPathSum(grid) {
>    const m = grid.length, n = grid[0].length;
>    const dp = Array.from({ length: m }, () => Array(n).fill(0));
>    
>    dp[0][0] = grid[0][0];
>    for (let i = 1; i < m; i++) dp[i][0] = dp[i - 1][0] + grid[i][0];
>    for (let j = 1; j < n; j++) dp[0][j] = dp[0][j - 1] + grid[0][j];
>    
>    for (let i = 1; i < m; i++) {
>        for (let j = 1; j < n; j++) {
>            dp[i][j] = Math.min(dp[i - 1][j], dp[i][j - 1]) + grid[i][j];
>        }
>    }
>    return dp[m - 1][n - 1];
>}
>```

### Maximal Square

```python
def maximal_square(matrix):
    if not matrix:
        return 0

    m, n = len(matrix), len(matrix[0])
    dp = [[0] * n for _ in range(m)]
    max_side = 0

    for i in range(m):
        for j in range(n):
            if matrix[i][j] == '1':
                if i == 0 or j == 0:
                    dp[i][j] = 1
                else:
                    dp[i][j] = min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1]) + 1
                max_side = max(max_side, dp[i][j])

    return max_side * max_side
```

**State**: `dp[i][j]` = side length of largest square with bottom-right at (i,j)

## Interval DP

### Longest Palindromic Subsequence

```python
def longest_palindrome_subseq(s):
    n = len(s)
    dp = [[0] * n for _ in range(n)]

    for i in range(n):
        dp[i][i] = 1  # Single char is palindrome

    for length in range(2, n + 1):
        for i in range(n - length + 1):
            j = i + length - 1
            if s[i] == s[j]:
                dp[i][j] = dp[i+1][j-1] + 2
            else:
                dp[i][j] = max(dp[i+1][j], dp[i][j-1])

    return dp[0][n-1]
```

**State**: `dp[i][j]` = LPS length for substring s[i..j]
**Order**: Process by increasing substring length

## Space Optimization

2D → 1D when current row only depends on previous row:

```python
# Original 2D
dp = [[0] * (n + 1) for _ in range(m + 1)]
for i in range(1, m + 1):
    for j in range(1, n + 1):
        dp[i][j] = dp[i-1][j] + dp[i][j-1]

# Optimized 1D
dp = [0] * (n + 1)
for i in range(1, m + 1):
    for j in range(1, n + 1):
        dp[j] = dp[j] + dp[j-1]  # dp[j] is prev row, dp[j-1] is current row
```

## Common Pitfalls

1. **Iteration order**: For interval DP, must process smaller intervals first
2. **Index mapping**: dp array indices vs string indices (often off by 1)
3. **Base case boundaries**: Edge rows/columns often need special handling
4. **Space optimization breaking**: Some recurrences need both (i-1, j-1), making 1D tricky

## Key Interview Problems

| Problem | Pattern | Difficulty | LeetCode Link |
| --------- | --------- | ------------ | --- |
| Unique Paths | Grid traversal | Medium | [Link](https://leetcode.com/problems/unique-paths/) |
| Minimum Path Sum | Grid optimization | Medium | [Link](https://leetcode.com/problems/minimum-path-sum/) |
| Longest Common Subsequence | Two strings | Medium | [Link](https://leetcode.com/problems/longest-common-subsequence/) |
| Edit Distance | Two strings | Medium | [Link](https://leetcode.com/problems/edit-distance/) |
| Longest Palindromic Subsequence | Interval | Medium | [Link](https://leetcode.com/problems/longest-palindromic-subsequence/) |
| Maximal Square | Matrix | Medium | [Link](https://leetcode.com/problems/maximal-square/) |
| Interleaving String | Two strings | Medium | [Link](https://leetcode.com/problems/interleaving-string/) |
