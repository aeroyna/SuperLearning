# Unbounded Knapsack

In the Unbounded Knapsack problem, each item can be selected **unlimited** times.

## Problem Statement
Given `n` items, each with a `weight` and a `value`, determine the maximum value you can achieve by selecting items such that their total weight does not exceed a given `capacity`. You may reuse items as many times as you like.

## Dynamic Programming Approach

### State
`dp[w]` = Max profit with capacity `w`.

### Transitions
For each item `i` with weight `wt` and value `val`:
We can include item `i` even if we have already included it.
`dp[w] = max(dp[w], val + dp[w - wt])`

### Space Optimization
Unlike 0/1 Knapsack, where we iterated backwards to avoid re-using items in the same step, here we iterate **forwards**.
Iterating forwards means when we compute `dp[w]`, `dp[w - wt]` might already reflect the inclusion of the *current* item, effectively allowing us to add it again.

## Implementation

>[!example]- C++
>```cpp
>int unboundedKnapsack(vector<int>& weights, vector<int>& values, int capacity) {
>    int n = weights.size();
>    vector<int> dp(capacity + 1, 0);
>
>    for (int w = 0; w <= capacity; w++) {
>        for (int i = 0; i < n; i++) {
>            if (weights[i] <= w) {
>                dp[w] = max(dp[w], values[i] + dp[w - weights[i]]);
>            }
>        }
>    }
>    // Alternative loop order (Item outer, Weight inner - same result)
>    /*
>    for (int i = 0; i < n; i++) {
>        for (int w = weights[i]; w <= capacity; w++) {
>            dp[w] = max(dp[w], values[i] + dp[w - weights[i]]);
>        }
>    }
>    */
>    return dp[capacity];
>}
>```

>[!example]- Java
>```java
>public int unboundedKnapsack(int[] weights, int[] values, int capacity) {
>    int[] dp = new int[capacity + 1];
>
>    // Iterate through all capacities
>    for (int w = 0; w <= capacity; w++) {
>        // Try every item for current capacity
>        for (int i = 0; i < weights.length; i++) {
>            if (weights[i] <= w) {
>                dp[w] = Math.max(dp[w], values[i] + dp[w - weights[i]]);
>            }
>        }
>    }
>    return dp[capacity];
>}
>```

>[!example]- Python
>```python
>def unbounded_knapsack(weights, values, capacity):
>    dp = [0] * (capacity + 1)
>
>    # Outer loop items, inner loop weights (forward)
>    for i in range(len(weights)):
>        for w in range(weights[i], capacity + 1):
>            dp[w] = max(dp[w], values[i] + dp[w - weights[i]])
>            
>    return dp[capacity]
>```

>[!example]- JavaScript
>```javascript
>function unboundedKnapsack(weights, values, capacity) {
>    const dp = new Array(capacity + 1).fill(0);
>
>    for (let i = 0; i < weights.length; i++) {
>        for (let w = weights[i]; w <= capacity; w++) {
>            dp[w] = Math.max(dp[w], values[i] + dp[w - weights[i]]);
>        }
>    }
>    return dp[capacity];
>}
>```

## Key Differences from 0/1 Knapsack
| Feature | 0/1 Knapsack | Unbounded Knapsack |
| :--- | :--- | :--- |
| **Item Use** | Max once | Unlimited |
| **Inner Loop** | **Backwards** (capacity down to weight) | **Forwards** (weight up to capacity) |
| **Common Problem** | Partition Equal Subset Sum | Coin Change |

## Complexity
- **Time**: `O(N * W)`
- **Space**: `O(W)`
