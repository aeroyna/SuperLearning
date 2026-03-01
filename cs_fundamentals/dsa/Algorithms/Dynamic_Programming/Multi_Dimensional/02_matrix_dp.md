## Matrix DP

Matrix DP is a common subset of 2D Dynamic Programming where the problem is set on a grid or matrix. The state `dp[i][j]` typically represents the solution to a subproblem concerning the grid cell at `(i, j)`. These problems often involve finding a path from a starting point (like the top-left corner) to an ending point (like the bottom-right corner).

### The General Pattern
- **State `dp[i][j]`**: Represents a calculated value for the cell `(i, j)`, such as the number of paths to reach it, or the minimum cost to reach it.
- **Recurrence Relation**: The value of `dp[i][j]` is usually a function of its adjacent cells, most commonly the one above (`dp[i-1][j]`) and the one to the left (`dp[i][j-1]`), as movement is often restricted to "down" and "right".
- **Initialization**: The base cases are typically the starting cell `dp[0][0]` and sometimes the entire first row and first column.

### Classic Example: Unique Paths (LeetCode #62)
**Problem**: A robot is located at the top-left corner of a `m x n` grid. The robot can only move either **down** or **right** at any point in time. The robot is trying to reach the bottom-right corner. How many possible unique paths are there?

**DP Approach**:
- **State**: `dp[i][j]` = the number of unique paths to reach the cell `(i, j)`.
- **Recurrence Relation**: To get to cell `(i, j)`, the robot must have come from either the cell directly above, `(i-1, j)`, or the cell directly to the left, `(i, j-1)`. Therefore, the total number of unique paths to `(i, j)` is the sum of the paths to these two cells.
  `dp[i][j] = dp[i-1][j] + dp[i][j-1]`
- **Base Cases**:
  - The first row (`dp[0][j]`) can only be reached by moving right from the start. There is only 1 way to reach each cell in the first row. So, `dp[0][j] = 1` for all `j`.
  - Similarly, the first column (`dp[i][0]`) can only be reached by moving down. There is only 1 way to reach each cell in the first column. So, `dp[i][0] = 1` for all `i`.

#### Implementation

>[!example]- C++
>```cpp
>class Solution {
>public:
>    int uniquePaths(int m, int n) {
>        // Initialize an m x n DP table with 1s, handling the base cases implicitly
>        vector<vector<int>> dp(m, vector<int>(n, 1));
>
>        // Fill the rest of the table using the recurrence relation
>        for (int i = 1; i < m; i++) {
>            for (int j = 1; j < n; j++) {
>                dp[i][j] = dp[i-1][j] + dp[i][j-1];
>            }
>        }
>
>        // The answer is the number of paths to the bottom-right cell
>        return dp[m-1][n-1];
>    }
>};
>```

>[!example]- Java
>```java
>class Solution {
>    public int uniquePaths(int m, int n) {
>        // Initialize an m x n DP table with 1s, handling the base cases implicitly
>        int[][] dp = new int[m][n];
>        for (int i = 0; i < m; i++) {
>            for (int j = 0; j < n; j++) {
>                dp[i][j] = 1;
>            }
>        }
>
>        // Fill the rest of the table using the recurrence relation
>        for (int i = 1; i < m; i++) {
>            for (int j = 1; j < n; j++) {
>                dp[i][j] = dp[i-1][j] + dp[i][j-1];
>            }
>        }
>
>        // The answer is the number of paths to the bottom-right cell
>        return dp[m-1][n-1];
>    }
>}
>```

>[!example]- Python
>```python
>def unique_paths(m: int, n: int) -> int:
>    # Initialize an m x n DP table with 1s, handling the base cases implicitly
>    dp = [[1] * n for _ in range(m)]
>
>    # Fill the rest of the table using the recurrence relation
>    for i in range(1, m):
>        for j in range(1, n):
>            dp[i][j] = dp[i-1][j] + dp[i][j-1]
>
>    # The answer is the number of paths to the bottom-right cell
>    return dp[m-1][n-1]
>```

>[!example]- JavaScript
>```javascript
>function uniquePaths(m, n) {
>    // Initialize an m x n DP table with 1s, handling the base cases implicitly
>    const dp = Array(m).fill(0).map(() => Array(n).fill(1));
>
>    // Fill the rest of the table using the recurrence relation
>    for (let i = 1; i < m; i++) {
>        for (let j = 1; j < n; j++) {
>            dp[i][j] = dp[i-1][j] + dp[i][j-1];
>        }
>    }
>
>    // The answer is the number of paths to the bottom-right cell
>    return dp[m-1][n-1];
>}
>```

### Space Optimization
For this pattern of problems where `dp[i][j]` only depends on the previous row and the current row, the space complexity can be optimized from O(m*n) to O(n). Instead of storing the entire matrix, you only need to store the DP values for the previous row to compute the current row. You can do this with a single 1D array. This is a common follow-up question in interviews.
