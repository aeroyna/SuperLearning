# Common DP Patterns

Certain DP patterns appear frequently across different problems. Recognizing these patterns helps identify the right approach quickly and structure solutions correctly.

## Overview

Key patterns covered:
- DP on Strings
- State Machine DP
- Knapsack variants
- Decision DP (take/skip)

## Topics

- [16.5.1 DP on Strings](01_dp_on_strings.md)
- [16.6.1 State Machine DP](02_state_machine_dp.md)

## Pattern 1: Knapsack

### 0/1 Knapsack

Each item can be taken once or not at all.

>[!example]- C++
>```cpp
>int knapsack01(vector<int>& weights, vector<int>& values, int capacity) {
>    int n = weights.size();
>    vector<vector<int>> dp(n + 1, vector<int>(capacity + 1, 0));
>    
>    for (int i = 1; i <= n; i++) {
>        for (int w = 0; w <= capacity; w++) {
>            dp[i][w] = dp[i - 1][w]; // Don't take
>            if (weights[i - 1] <= w) {
>                dp[i][w] = max(dp[i][w], dp[i - 1][w - weights[i - 1]] + values[i - 1]);
>            }
>        }
>    }
>    return dp[n][capacity];
>}
>```

>[!example]- Java
>```java
>public int knapsack01(int[] weights, int[] values, int capacity) {
>    int n = weights.length;
>    int[][] dp = new int[n + 1][capacity + 1];
>    
>    for (int i = 1; i <= n; i++) {
>        for (int w = 0; w <= capacity; w++) {
>            dp[i][w] = dp[i - 1][w]; // Don't take
>            if (weights[i - 1] <= w) {
>                dp[i][w] = Math.max(dp[i][w], dp[i - 1][w - weights[i - 1]] + values[i - 1]);
>            }
>        }
>    }
>    return dp[n][capacity];
>}
>```

>[!example]- Python
>```python
>def knapsack_01(weights, values, capacity):
>    n = len(weights)
>    dp = [[0] * (capacity + 1) for _ in range(n + 1)]
>
>    for i in range(1, n + 1):
>        for w in range(capacity + 1):
>            dp[i][w] = dp[i-1][w]  # Don't take
>            if weights[i-1] <= w:
>                dp[i][w] = max(dp[i][w],
>                              dp[i-1][w - weights[i-1]] + values[i-1])
>
>    return dp[n][capacity]
>```

>[!example]- JavaScript
>```javascript
>function knapsack01(weights, values, capacity) {
>    const n = weights.length;
>    const dp = Array.from({ length: n + 1 }, () => Array(capacity + 1).fill(0));
>    
>    for (let i = 1; i <= n; i++) {
>        for (let w = 0; w <= capacity; w++) {
>            dp[i][w] = dp[i - 1][w]; // Don't take
>            if (weights[i - 1] <= w) {
>                dp[i][w] = Math.max(dp[i][w], dp[i - 1][w - weights[i - 1]] + values[i - 1]);
>            }
>        }
>    }
>    return dp[n][capacity];
>}
>```

**State**: `dp[i][w]` = max value using items 0..i-1 with capacity w
**Decision**: Take item i (if fits) or skip

### Unbounded Knapsack

Each item can be taken unlimited times.

>[!example]- C++
>```cpp
>int knapsackUnbounded(vector<int>& weights, vector<int>& values, int capacity) {
>    vector<int> dp(capacity + 1, 0);
>    
>    for (int w = 1; w <= capacity; w++) {
>        for (int i = 0; i < weights.size(); i++) {
>            if (weights[i] <= w) {
>                dp[w] = max(dp[w], dp[w - weights[i]] + values[i]);
>            }
>        }
>    }
>    return dp[capacity];
>}
>```

>[!example]- Java
>```java
>public int knapsackUnbounded(int[] weights, int[] values, int capacity) {
>    int[] dp = new int[capacity + 1];
>    
>    for (int w = 1; w <= capacity; w++) {
>        for (int i = 0; i < weights.length; i++) {
>            if (weights[i] <= w) {
>                dp[w] = Math.max(dp[w], dp[w - weights[i]] + values[i]);
>            }
>        }
>    }
>    return dp[capacity];
>}
>```

>[!example]- Python
>```python
>def knapsack_unbounded(weights, values, capacity):
>    dp = [0] * (capacity + 1)
>
>    for w in range(1, capacity + 1):
>        for i in range(len(weights)):
>            if weights[i] <= w:
>                dp[w] = max(dp[w], dp[w - weights[i]] + values[i])
>
>    return dp[capacity]
>```

>[!example]- JavaScript
>```javascript
>function knapsackUnbounded(weights, values, capacity) {
>    const dp = new Array(capacity + 1).fill(0);
>    
>    for (let w = 1; w <= capacity; w++) {
>        for (let i = 0; i < weights.length; i++) {
>            if (weights[i] <= w) {
>                dp[w] = Math.max(dp[w], dp[w - weights[i]] + values[i]);
>            }
>        }
>    }
>    return dp[capacity];
>}
>```

**Key difference**: Use `dp[w - weights[i]]` (same row), not `dp[i-1][w - weights[i]]`

### Subset Sum

Special case: values = weights, goal is exact sum.

```python
def can_partition(nums, target):
    dp = [False] * (target + 1)
    dp[0] = True

    for num in nums:
        for j in range(target, num - 1, -1):  # Reverse to avoid reuse
            dp[j] = dp[j] or dp[j - num]

    return dp[target]
```

## Pattern 2: State Machine DP

Model problem states and transitions explicitly.

### Best Time to Buy and Sell Stock with Cooldown

States: holding, not holding (sold), cooldown

```python
def max_profit(prices):
    if len(prices) <= 1:
        return 0

    # hold[i] = max profit on day i while holding stock
    # sold[i] = max profit on day i after just selling
    # rest[i] = max profit on day i doing nothing (cooldown)

    hold = -prices[0]
    sold = 0
    rest = 0

    for i in range(1, len(prices)):
        prev_hold = hold
        hold = max(hold, rest - prices[i])  # Keep holding or buy
        rest = max(rest, sold)               # Rest or continue rest
        sold = prev_hold + prices[i]         # Sell

    return max(sold, rest)
```

**State machine visualization**:
```
        buy            sell
rest ────────→ hold ────────→ sold
  ↑                              │
  │            rest              │
  └──────────────────────────────┘
```

### Stock with Transaction Fee

```python
def max_profit(prices, fee):
    hold = -prices[0]
    cash = 0

    for price in prices[1:]:
        hold = max(hold, cash - price)
        cash = max(cash, hold + price - fee)

    return cash
```

## Pattern 3: Decision DP (Take/Skip)

At each position, decide whether to include or exclude.

### House Robber Pattern

```python
def solve(nums):
    # At each index: take (and skip previous) or skip
    take = 0
    skip = 0

    for num in nums:
        new_take = skip + num  # Take current, must have skipped previous
        new_skip = max(take, skip)  # Skip current
        take, skip = new_take, new_skip

    return max(take, skip)
```

### Delete and Earn

Transform to House Robber by grouping by value.

```python
def delete_and_earn(nums):
    if not nums:
        return 0

    max_val = max(nums)
    points = [0] * (max_val + 1)
    for num in nums:
        points[num] += num

    # Now it's House Robber on points array
    take = skip = 0
    for point in points:
        take, skip = skip + point, max(take, skip)

    return max(take, skip)
```

## Pattern 4: Counting DP

Count number of ways (not optimize).

### Decode Ways

```python
def num_decodings(s):
    if not s or s[0] == '0':
        return 0

    n = len(s)
    dp = [0] * (n + 1)
    dp[0] = 1
    dp[1] = 1

    for i in range(2, n + 1):
        # Single digit (1-9)
        if s[i-1] != '0':
            dp[i] += dp[i-1]

        # Two digits (10-26)
        two_digit = int(s[i-2:i])
        if 10 <= two_digit <= 26:
            dp[i] += dp[i-2]

    return dp[n]
```

## Pattern Recognition Guide

| Problem Characteristic | Pattern | Example |
|----------------------|---------|---------|
| Items with weights/values | Knapsack | Coin Change |
| Distinct states with transitions | State Machine | Stock problems |
| Take/skip at each position | Decision DP | House Robber |
| Count arrangements | Counting DP | Decode Ways |
| Two sequences | 2D String DP | LCS, Edit Distance |
| Substring/interval | Interval DP | Palindrome |

## Common Pitfalls

1. **Wrong iteration direction**: 0/1 knapsack needs reverse iteration in 1D
2. **Missing state**: State machine needs all possible states
3. **Incorrect transition**: Carefully map problem constraints to transitions
4. **Overflow in counting**: Use modulo for large counts

## Key Interview Problems

| Problem | Pattern | Difficulty | LeetCode Link |
| --------- | --------- | ------------ | --- |
| Partition Equal Subset Sum | 0/1 Knapsack | Medium | [Link](https://leetcode.com/problems/partition-equal-subset-sum/) |
| Coin Change | Unbounded Knapsack | Medium | [Link](https://leetcode.com/problems/coin-change/) |
| Best Time to Buy and Sell Stock III | State Machine | Hard | [Link](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iii/) |
| House Robber II | Decision DP (circular) | Medium | [Link](https://leetcode.com/problems/house-robber-ii/) |
| Decode Ways | Counting | Medium | [Link](https://leetcode.com/problems/decode-ways/) |
| Target Sum | Counting Knapsack | Medium | [Link](https://leetcode.com/problems/target-sum/) |
