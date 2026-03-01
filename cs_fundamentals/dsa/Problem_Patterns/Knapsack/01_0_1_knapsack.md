# 0/1 Knapsack

In the 0/1 Knapsack problem, each item can be selected at most **once**.

## Problem Statement
Given `n` items, each with a `weight` and a `value`, determine the maximum value you can achieve by selecting a subset of items such that their total weight does not exceed a given `capacity`.

## Dynamic Programming Approach

### State
`dp[i][w]` = Max profit using a subset of the first `i` items with capacity `w`.

### Transitions
For each item `i` with weight `wt` and value `val`:
1.  **Exclude item `i`**: We carry forward the profit from the previous state: `dp[i-1][w]`.
2.  **Include item `i`**: We add `val` to the profit achievable with the remaining capacity: `val + dp[i-1][w - wt]`. This is valid only if `wt <= w`.

`dp[i][w] = max(dp[i-1][w], val + dp[i-1][w - wt])`

### Space Optimization
Since `dp[i]` only depends on `dp[i-1]`, we can reduce the space complexity from `O(N * W)` to `O(W)` using a 1D array.
Crucially, when using a 1D array, we must iterate **backwards** through the weights to ensure we don't use the same item multiple times for the same item index `i`.

## Implementation (Space Optimized)

>[!example]- C++
>```cpp
>int knapsack(vector<int>& weights, vector<int>& values, int capacity) {
>    int n = weights.size();
>    vector<int> dp(capacity + 1, 0);
>
>    for (int i = 0; i < n; i++) {
>        // Iterate backwards to avoid using the same item twice in one step
>        for (int w = capacity; w >= weights[i]; w--) {
>            dp[w] = max(dp[w], values[i] + dp[w - weights[i]]);
>        }
>    }
>    return dp[capacity];
>}
>```

>[!example]- Java
>```java
>public int knapsack(int[] weights, int[] values, int capacity) {
>    int n = weights.length;
>    int[] dp = new int[capacity + 1];
>
>    for (int i = 0; i < n; i++) {
>        // Iterate backwards
>        for (int w = capacity; w >= weights[i]; w--) {
>            dp[w] = Math.max(dp[w], values[i] + dp[w - weights[i]]);
>        }
>    }
>    return dp[capacity];
>}
>```

>[!example]- Python
>```python
>def knapsack(weights, values, capacity):
>    n = len(weights)
>    dp = [0] * (capacity + 1)
>
>    for i in range(n):
>        # Iterate backwards
>        for w in range(capacity, weights[i] - 1, -1):
>            dp[w] = max(dp[w], values[i] + dp[w - weights[i]])
>            
>    return dp[capacity]
>```

>[!example]- JavaScript
>```javascript
>function knapsack(weights, values, capacity) {
>    const n = weights.length;
>    const dp = new Array(capacity + 1).fill(0);
>
>    for (let i = 0; i < n; i++) {
>        // Iterate backwards
>        for (let w = capacity; w >= weights[i]; w--) {
>            dp[w] = Math.max(dp[w], values[i] + dp[w - weights[i]]);
>        }
>    }
>    return dp[capacity];
>}
>```

## Complexity
- **Time**: `O(N * W)`
- **Space**: `O(W)`
