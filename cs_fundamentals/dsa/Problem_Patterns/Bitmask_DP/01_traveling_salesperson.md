# Traveling Salesperson (TSP)

The Traveling Salesperson Problem (TSP) is the most classic example of Bitmask DP.

## Problem Statement
Given a set of cities and the distances between every pair of cities, find the shortest possible route that visits every city exactly once and returns to the origin city.

Constraints usually involve `N <= 20`.

## Approach

### State
`dp[mask][last_city]`
- `mask`: Bitmask representing the set of visited cities.
- `last_city`: The index of the last city visited.

### Transitions
To compute `dp[mask][u]`:
We must have come from some previous state `dp[prev_mask][v]`, where:
- `u` is in `mask`.
- `prev_mask` is `mask` without `u`.
- `v` is any city in `prev_mask`.

`dp[mask][u] = min(dp[prev_mask][v] + dist[v][u])`

## Implementation

>[!example]- C++
>```cpp
>int tsp(vector<vector<int>>& dist) {
>    int n = dist.size();
>    int ALL_VISITED = (1 << n) - 1;
>    vector<vector<int>> dp(1 << n, vector<int>(n, 1e9));
>    
>    // Base case: Start at city 0
>    dp[1][0] = 0;
>    
>    for (int mask = 1; mask < (1 << n); mask++) {
>        for (int u = 0; u < n; u++) {
>            // If u is in current subset
>            if ((mask >> u) & 1) {
>                // Try moving to next city v
>                for (int v = 0; v < n; v++) {
>                    if (!((mask >> v) & 1)) { // If v not visited
>                        int nextMask = mask | (1 << v);
>                        dp[nextMask][v] = min(dp[nextMask][v], dp[mask][u] + dist[u][v]);
>                    }
>                }
>            }
>        }
>    }
>    
>    // Return to start (0) from any end node
>    int ans = 1e9;
>    for (int i = 1; i < n; i++) {
>        ans = min(ans, dp[ALL_VISITED][i] + dist[i][0]);
>    }
>    return ans;
>}
>```

>[!example]- Java
>```java
>public int tsp(int[][] dist) {
>    int n = dist.length;
>    int ALL_VISITED = (1 << n) - 1;
>    int[][] dp = new int[1 << n][n];
>    
>    for (int[] row : dp) Arrays.fill(row, 1000000000);
>    
>    // Base case: Start at city 0
>    dp[1][0] = 0;
>    
>    for (int mask = 1; mask < (1 << n); mask++) {
>        for (int u = 0; u < n; u++) {
>            // If u is in current subset
>            if (((mask >> u) & 1) == 1) {
>                // Try moving to next city v
>                for (int v = 0; v < n; v++) {
>                    if (((mask >> v) & 1) == 0) { // If v not visited
>                        int nextMask = mask | (1 << v);
>                        dp[nextMask][v] = Math.min(dp[nextMask][v], dp[mask][u] + dist[u][v]);
>                    }
>                }
>            }
>        }
>    }
>    
>    int ans = 1000000000;
>    for (int i = 1; i < n; i++) {
>        ans = Math.min(ans, dp[ALL_VISITED][i] + dist[i][0]);
>    }
>    return ans;
>}
>```

>[!example]- Python
>```python
>def tsp(dist):
>    n = len(dist)
>    ALL_VISITED = (1 << n) - 1
>    # dp[mask][last_city]
>    dp = [[float('inf')] * n for _ in range(1 << n)]
>    
>    # Base case: Start at city 0
>    dp[1][0] = 0
>    
>    for mask in range(1, 1 << n):
>        for u in range(n):
>            # If u is in current subset
>            if (mask >> u) & 1:
>                # Try moving to next city v
>                for v in range(n):
>                    if not ((mask >> v) & 1): # If v not visited
>                        next_mask = mask | (1 << v)
>                        dp[next_mask][v] = min(dp[next_mask][v], dp[mask][u] + dist[u][v])
>                        
>    # Return to start (0)
>    ans = float('inf')
>    for i in range(1, n):
>        ans = min(ans, dp[ALL_VISITED][i] + dist[i][0])
>        
>    return ans
>```

>[!example]- JavaScript
>```javascript
>function tsp(dist) {
>    const n = dist.length;
>    const ALL_VISITED = (1 << n) - 1;
>    // dp[mask][last_city]
>    const dp = Array.from({ length: 1 << n }, () => new Array(n).fill(Infinity));
>    
>    // Base case: Start at city 0
>    dp[1][0] = 0;
>    
>    for (let mask = 1; mask < (1 << n); mask++) {
>        for (let u = 0; u < n; u++) {
>            // If u is in current subset
>            if ((mask >> u) & 1) {
>                // Try moving to next city v
>                for (let v = 0; v < n; v++) {
>                    if (!((mask >> v) & 1)) { // If v not visited
>                        const nextMask = mask | (1 << v);
>                        dp[nextMask][v] = Math.min(dp[nextMask][v], dp[mask][u] + dist[u][v]);
>                    }
>                }
>            }
>        }
>    }
>    
>    let ans = Infinity;
>    for (let i = 1; i < n; i++) {
>        ans = Math.min(ans, dp[ALL_VISITED][i] + dist[i][0]);
>    }
>    return ans;
>}
>```
