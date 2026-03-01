## 2D Dynamic Programming Problems

2D DP problems involve a state that is defined by two variables, `dp[i][j]`. This usually occurs when you are working with two inputs (e.g., two strings) or when the problem requires tracking two different parameters. The DP table becomes a 2D matrix.

### The General Pattern
- **State `dp[i][j]`**: Represents the solution to a subproblem concerning the first `i` elements of input 1 and the first `j` elements of input 2.
- **Recurrence Relation**: `dp[i][j]` is typically computed based on the values of its neighbors in the DP table: `dp[i-1][j]`, `dp[i][j-1]`, and often `dp[i-1][j-1]`.
- **Initialization**: The first row and first column of the DP table are usually initialized as the base cases.

### Classic Example: Longest Common Subsequence (LCS)
**Problem**: Given two strings, `text1` and `text2`, find the length of their longest common subsequence. A subsequence is a sequence that can be derived from another sequence by deleting some or no elements without changing the order of the remaining elements. (LeetCode #1143)

**Example**: `text1 = "abcde"`, `text2 = "ace"`. The LCS is `"ace"`, with a length of 3.

**DP Approach**:
- **State**: `dp[i][j]` will be the length of the LCS of the prefixes `text1[0...i-1]` and `text2[0...j-1]`. We use a DP table of size `(len(text1)+1) x (len(text2)+1)` to handle empty string base cases easily.

- **Recurrence Relation**: We compare the characters at `text1[i-1]` and `text2[j-1]`.
  1.  **If the characters match (`text1[i-1] == text2[j-1]`)**: The character is part of the LCS. We add 1 to the length of the LCS of the prefixes before it.
      `dp[i][j] = 1 + dp[i-1][j-1]`
  2.  **If the characters do not match**: The common subsequence does not include one of these characters. We must take the best result (the maximum length) from the two possible preceding subproblems:
      - The LCS of `text1[0...i-2]` and `text2[0...j-1]` (i.e., `dp[i-1][j]`).
      - The LCS of `text1[0...i-1]` and `text2[0...j-2]` (i.e., `dp[i][j-1]`).
      `dp[i][j] = max(dp[i-1][j], dp[i][j-1])`

- **Base Cases**: `dp[0][j] = 0` and `dp[i][0] = 0` for all `i, j`. The LCS of any string with an empty string is 0. This is handled by initializing the DP table with zeros.

#### Implementation

>[!example]- C++
>```cpp
>class Solution {
>public:
>    int longestCommonSubsequence(string text1, string text2) {
>        int m = text1.length(), n = text2.length();
>
>        // Initialize DP table with zeros. dp[i][j] corresponds to text1[:i] and text2[:j]
>        vector<vector<int>> dp(m + 1, vector<int>(n + 1, 0));
>
>        for (int i = 1; i <= m; i++) {
>            for (int j = 1; j <= n; j++) {
>                // If characters match
>                if (text1[i-1] == text2[j-1]) {
>                    dp[i][j] = 1 + dp[i-1][j-1];
>                }
>                // If they don't match
>                else {
>                    dp[i][j] = max(dp[i-1][j], dp[i][j-1]);
>                }
>            }
>        }
>
>        // The answer is in the bottom-right cell
>        return dp[m][n];
>    }
>};
>```

>[!example]- Java
>```java
>class Solution {
>    public int longestCommonSubsequence(String text1, String text2) {
>        int m = text1.length(), n = text2.length();
>
>        // Initialize DP table with zeros. dp[i][j] corresponds to text1[:i] and text2[:j]
>        int[][] dp = new int[m + 1][n + 1];
>
>        for (int i = 1; i <= m; i++) {
>            for (int j = 1; j <= n; j++) {
>                // If characters match
>                if (text1.charAt(i-1) == text2.charAt(j-1)) {
>                    dp[i][j] = 1 + dp[i-1][j-1];
>                }
>                // If they don't match
>                else {
>                    dp[i][j] = Math.max(dp[i-1][j], dp[i][j-1]);
>                }
>            }
>        }
>
>        // The answer is in the bottom-right cell
>        return dp[m][n];
>    }
>}
>```

>[!example]- Python
>```python
>def longest_common_subsequence(text1: str, text2: str) -> int:
>    m, n = len(text1), len(text2)
>
>    # Initialize DP table with zeros. dp[i][j] corresponds to text1[:i] and text2[:j]
>    dp = [[0] * (n + 1) for _ in range(m + 1)]
>
>    for i in range(1, m + 1):
>        for j in range(1, n + 1):
>            # If characters match
>            if text1[i-1] == text2[j-1]:
>                dp[i][j] = 1 + dp[i-1][j-1]
>            # If they don't match
>            else:
>                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
>
>    # The answer is in the bottom-right cell
>    return dp[m][n]
>```

>[!example]- JavaScript
>```javascript
>function longestCommonSubsequence(text1, text2) {
>    const m = text1.length, n = text2.length;
>
>    // Initialize DP table with zeros. dp[i][j] corresponds to text1[:i] and text2[:j]
>    const dp = Array(m + 1).fill(0).map(() => Array(n + 1).fill(0));
>
>    for (let i = 1; i <= m; i++) {
>        for (let j = 1; j <= n; j++) {
>            // If characters match
>            if (text1[i-1] === text2[j-1]) {
>                dp[i][j] = 1 + dp[i-1][j-1];
>            }
>            // If they don't match
>            else {
>                dp[i][j] = Math.max(dp[i-1][j], dp[i][j-1]);
>            }
>        }
>    }
>
>    // The answer is in the bottom-right cell
>    return dp[m][n];
>}
>```

This pattern is fundamental and also applies to related problems like Edit Distance and Unbounded Knapsack.
