## DP on Strings

A significant number of Dynamic Programming problems involve comparing, matching, or transforming one or more strings. These problems are almost always solved using a 2D DP table where the state `dp[i][j]` represents a solution for the prefixes of the strings.

### The General Pattern
- **Input**: Two strings, `s1` and `s2`, with lengths `m` and `n`.
- **State**: `dp[i][j]` represents the solution for the subproblem concerning the prefix `s1[:i]` and `s2[:j]`. The DP table is typically of size `(m+1) x (n+1)` to accommodate empty string base cases.
- **Recurrence**: The core of the recurrence relation almost always involves two cases based on comparing `s1[i-1]` and `s2[j-1]`:
    1. **Characters Match**: If `s1[i-1] == s2[j-1]`, the solution for `dp[i][j]` often depends on the solution for the smaller subproblem `dp[i-1][j-1]`.
    2. **Characters Do Not Match**: If the characters are different, the solution `dp[i][j]` is usually derived from the solutions of the subproblems where one character is excluded, i.e., `dp[i-1][j]` and `dp[i][j-1]`.

This pattern is the backbone of several classic string problems.

### Classic Problems Using This Pattern

#### 1. Longest Common Subsequence (LCS)
- **Goal**: Find the length of the longest subsequence common to both strings.
- **Recurrence**:
  - If `s1[i-1] == s2[j-1]`: `dp[i][j] = 1 + dp[i-1][j-1]`
  - If `s1[i-1] != s2[j-1]`: `dp[i][j] = max(dp[i-1][j], dp[i][j-1])`

#### 2. Edit Distance
- **Goal**: Find the minimum number of operations (insert, delete, or substitute) required to convert `s1` to `s2`.
- **State**: `dp[i][j]` = the minimum distance between `s1[:i]` and `s2[:j]`.
- **Recurrence**:
  - If `s1[i-1] == s2[j-1]`: No operation needed for this character. `dp[i][j] = dp[i-1][j-1]`.
  - If `s1[i-1] != s2[j-1]`: We must perform one operation. We take the minimum of the three possibilities:
    - **Substitute**: `1 + dp[i-1][j-1]` (change `s1[i-1]` to `s2[j-1]`)
    - **Delete** (from s1): `1 + dp[i-1][j]` (delete `s1[i-1]`)
    - **Insert** (into s1): `1 + dp[i][j-1]` (insert `s2[j-1]`)
    `dp[i][j] = 1 + min(dp[i-1][j-1], dp[i-1][j], dp[i][j-1])`

#### 3. Distinct Subsequences
- **Goal**: Count how many times `s2` appears as a subsequence of `s1`.
- **State**: `dp[i][j]` = the number of distinct subsequences of `s1[:i]` that equal `s2[:j]`.
- **Recurrence**:
  - If `s1[i-1] != s2[j-1]`: The character from `s1` cannot be used to form the subsequence. The answer is the same as the subproblem without this character: `dp[i][j] = dp[i-1][j]`.
  - If `s1[i-1] == s2[j-1]`: The character from `s1` *can* be used. We have two sources of solutions:
    1.  The number of ways to form `s2[:j]` without using `s1[i-1]`, which is `dp[i-1][j]`.
    2.  The number of ways to form `s2[:j-1]` from `s1[:i-1]`, to which we now match `s1[i-1]` with `s2[j-1]`. This is `dp[i-1][j-1]`.
    `dp[i][j] = dp[i-1][j] + dp[i-1][j-1]`

Recognizing this `dp[i][j]` on `s1[:i]` and `s2[:j]` pattern is key to solving a whole family of string-based DP problems.
