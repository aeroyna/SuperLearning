# One-Dimensional Dynamic Programming

1D DP problems have a single state variable (usually an index) and build solutions by combining answers from previous states. These are the foundation for understanding more complex DP patterns.

## Overview

In 1D DP:
- State: `dp[i]` represents answer for subproblem at index i
- Transition: `dp[i]` depends on `dp[i-1]`, `dp[i-2]`, etc.
- Space optimization often possible: O(n) → O(1)

## Topics

- [16.2.1 1D DP Problems](01_1d_dp_problems.md)

## Framework

### Step 1: Define State

What does `dp[i]` represent?
- `dp[i]` = answer considering elements 0 to i
- `dp[i]` = answer ending at index i
- `dp[i]` = answer using exactly i elements

### Step 2: Find Recurrence

How does `dp[i]` relate to previous states?

### Step 3: Identify Base Cases

What are `dp[0]`, `dp[1]`, etc.?

### Step 4: Determine Order

Process states so dependencies are computed first.

## Classic 1D DP Problems

### Climbing Stairs

>[!example]- C++
>```cpp
>int climbStairs(int n) {
>    if (n <= 2) return n;
>    int prev2 = 1, prev1 = 2;
>    for (int i = 3; i <= n; i++) {
>        int current = prev1 + prev2;
>        prev2 = prev1;
>        prev1 = current;
>    }
>    return prev1;
>}
>```

>[!example]- Java
>```java
>public int climbStairs(int n) {
>    if (n <= 2) return n;
>    int prev2 = 1, prev1 = 2;
>    for (int i = 3; i <= n; i++) {
>        int current = prev1 + prev2;
>        prev2 = prev1;
>        prev1 = current;
>    }
>    return prev1;
>}
>```

>[!example]- Python
>```python
>def climb_stairs(n):
>    """Ways to climb n stairs taking 1 or 2 steps."""
>    if n <= 2:
>        return n
>
>    dp = [0] * (n + 1)
>    dp[1], dp[2] = 1, 2
>
>    for i in range(3, n + 1):
>        dp[i] = dp[i-1] + dp[i-2]  # From i-1 or i-2
>
>    return dp[n]
>
># Space-optimized
>def climb_stairs_optimized(n):
>    if n <= 2:
>        return n
>    prev2, prev1 = 1, 2
>    for _ in range(3, n + 1):
>        prev2, prev1 = prev1, prev2 + prev1
>    return prev1
>```

>[!example]- JavaScript
>```javascript
>function climbStairs(n) {
>    if (n <= 2) return n;
>    let prev2 = 1, prev1 = 2;
>    for (let i = 3; i <= n; i++) {
>        const current = prev1 + prev2;
>        prev2 = prev1;
>        prev1 = current;
>    }
>    return prev1;
>}
>```

**State**: `dp[i]` = ways to reach step i
**Recurrence**: `dp[i] = dp[i-1] + dp[i-2]`

### House Robber

>[!example]- C++
>```cpp
>int rob(vector<int>& nums) {
>    if (nums.empty()) return 0;
>    if (nums.size() == 1) return nums[0];
>    
>    int prev2 = nums[0];
>    int prev1 = max(nums[0], nums[1]);
>    
>    for (int i = 2; i < nums.size(); i++) {
>        int current = max(prev1, prev2 + nums[i]);
>        prev2 = prev1;
>        prev1 = current;
>    }
>    return prev1;
>}
>```

>[!example]- Java
>```java
>public int rob(int[] nums) {
>    if (nums.length == 0) return 0;
>    if (nums.length == 1) return nums[0];
>    
>    int prev2 = nums[0];
>    int prev1 = Math.max(nums[0], nums[1]);
>    
>    for (int i = 2; i < nums.length; i++) {
>        int current = Math.max(prev1, prev2 + nums[i]);
>        prev2 = prev1;
>        prev1 = current;
>    }
>    return prev1;
>}
>```

>[!example]- Python
>```python
>def rob(nums):
>    """Max sum without adjacent elements."""
>    if not nums:
>        return 0
>    if len(nums) == 1:
>        return nums[0]
>
>    dp = [0] * len(nums)
>    dp[0] = nums[0]
>    dp[1] = max(nums[0], nums[1])
>
>    for i in range(2, len(nums)):
>        dp[i] = max(dp[i-1], dp[i-2] + nums[i])
>
>    return dp[-1]
>```

>[!example]- JavaScript
>```javascript
>function rob(nums) {
>    if (nums.length === 0) return 0;
>    if (nums.length === 1) return nums[0];
>    
>    let prev2 = nums[0];
>    let prev1 = Math.max(nums[0], nums[1]);
>    
>    for (let i = 2; i < nums.length; i++) {
>        const current = Math.max(prev1, prev2 + nums[i]);
>        prev2 = prev1;
>        prev1 = current;
>    }
>    return prev1;
>}
>```

**State**: `dp[i]` = max profit considering houses 0 to i
**Recurrence**: `dp[i] = max(skip this house, rob this house)`

### Coin Change (Minimum Coins)

>[!example]- C++
>```cpp
>int coinChange(vector<int>& coins, int amount) {
>    vector<int> dp(amount + 1, amount + 1);
>    dp[0] = 0;
>    
>    for (int i = 1; i <= amount; i++) {
>        for (int coin : coins) {
>            if (coin <= i) {
>                dp[i] = min(dp[i], dp[i - coin] + 1);
>            }
>        }
>    }
>    return dp[amount] > amount ? -1 : dp[amount];
>}
>```

>[!example]- Java
>```java
>public int coinChange(int[] coins, int amount) {
>    int[] dp = new int[amount + 1];
>    Arrays.fill(dp, amount + 1);
>    dp[0] = 0;
>    
>    for (int i = 1; i <= amount; i++) {
>        for (int coin : coins) {
>            if (coin <= i) {
>                dp[i] = Math.min(dp[i], dp[i - coin] + 1);
>            }
>        }
>    }
>    return dp[amount] > amount ? -1 : dp[amount];
>}
>```

>[!example]- Python
>```python
>def coin_change(coins, amount):
>    """Minimum coins to make amount."""
>    dp = [float('inf')] * (amount + 1)
>    dp[0] = 0
>
>    for i in range(1, amount + 1):
>        for coin in coins:
>            if coin <= i:
>                dp[i] = min(dp[i], dp[i - coin] + 1)
>
>    return dp[amount] if dp[amount] != float('inf') else -1
>```

>[!example]- JavaScript
>```javascript
>function coinChange(coins, amount) {
>    const dp = new Array(amount + 1).fill(amount + 1);
>    dp[0] = 0;
>    
>    for (let i = 1; i <= amount; i++) {
>        for (const coin of coins) {
>            if (coin <= i) {
>                dp[i] = Math.min(dp[i], dp[i - coin] + 1);
>            }
>        }
>    }
>    return dp[amount] > amount ? -1 : dp[amount];
>}
>```

**State**: `dp[i]` = minimum coins to make amount i
**Recurrence**: `dp[i] = min(dp[i - coin] + 1)` for all valid coins

### Longest Increasing Subsequence

>[!example]- C++
>```cpp
>int lengthOfLIS(vector<int>& nums) {
>    if (nums.empty()) return 0;
>    vector<int> dp(nums.size(), 1);
>    int maxLen = 1;
>    
>    for (int i = 1; i < nums.size(); i++) {
>        for (int j = 0; j < i; j++) {
>            if (nums[j] < nums[i]) {
>                dp[i] = max(dp[i], dp[j] + 1);
>            }
>        }
>        maxLen = max(maxLen, dp[i]);
>    }
>    return maxLen;
>}
>```

>[!example]- Java
>```java
>public int lengthOfLIS(int[] nums) {
>    if (nums.length == 0) return 0;
>    int[] dp = new int[nums.length];
>    Arrays.fill(dp, 1);
>    int maxLen = 1;
>    
>    for (int i = 1; i < nums.length; i++) {
>        for (int j = 0; j < i; j++) {
>            if (nums[j] < nums[i]) {
>                dp[i] = Math.max(dp[i], dp[j] + 1);
>            }
>        }
>        maxLen = Math.max(maxLen, dp[i]);
>    }
>    return maxLen;
>}
>```

>[!example]- Python
>```python
>def length_of_lis(nums):
>    """Length of longest strictly increasing subsequence."""
>    if not nums:
>        return 0
>
>    dp = [1] * len(nums)  # Each element is LIS of length 1
>
>    for i in range(1, len(nums)):
>        for j in range(i):
>            if nums[j] < nums[i]:
>                dp[i] = max(dp[i], dp[j] + 1)
>
>    return max(dp)
>```

>[!example]- JavaScript
>```javascript
>function lengthOfLIS(nums) {
>    if (nums.length === 0) return 0;
>    const dp = new Array(nums.length).fill(1);
>    let maxLen = 1;
>    
>    for (let i = 1; i < nums.length; i++) {
>        for (let j = 0; j < i; j++) {
>            if (nums[j] < nums[i]) {
>                dp[i] = Math.max(dp[i], dp[j] + 1);
>            }
>        }
>        maxLen = Math.max(maxLen, dp[i]);
>    }
>    return maxLen;
>}
>```

**State**: `dp[i]` = LIS length ending at index i
**Recurrence**: `dp[i] = max(dp[j] + 1)` for all j < i where nums[j] < nums[i]

**O(n log n) optimization using binary search exists.**

### Word Break

```python
def word_break(s, word_dict):
    """Can s be segmented into dictionary words?"""
    word_set = set(word_dict)
    dp = [False] * (len(s) + 1)
    dp[0] = True  # Empty string

    for i in range(1, len(s) + 1):
        for j in range(i):
            if dp[j] and s[j:i] in word_set:
                dp[i] = True
                break

    return dp[len(s)]
```

**State**: `dp[i]` = can s[0:i] be segmented
**Recurrence**: `dp[i] = any(dp[j] and s[j:i] in dict)`

## Space Optimization

When `dp[i]` only depends on last k states, reduce space to O(k):

```python
# From O(n) space
dp = [0] * n
for i in range(2, n):
    dp[i] = dp[i-1] + dp[i-2]

# To O(1) space
prev2, prev1 = dp[0], dp[1]
for i in range(2, n):
    prev2, prev1 = prev1, prev2 + prev1
```

## Common Patterns

| Pattern | State Definition | Example |
|---------|-----------------|---------|
| Linear scan | dp[i] = answer for 0..i | House Robber |
| Ending at i | dp[i] = answer ending at i | LIS |
| Starting at i | dp[i] = answer from i to end | Jump Game |
| Amount/target | dp[i] = answer for target i | Coin Change |

## Common Pitfalls

1. **Off-by-one in array size**: If dp[i] represents i elements, need size n+1
2. **Wrong base case**: Carefully define dp[0] meaning
3. **Missing states**: Ensure all needed states are computed before use
4. **Not considering "skip" option**: Many problems have "don't take" as a choice

## Key Interview Problems

| Problem | Pattern | Difficulty | LeetCode Link |
| --------- | --------- | ------------ | --- |
| Climbing Stairs | Fibonacci-like | Easy | [Link](https://leetcode.com/problems/climbing-stairs/) |
| House Robber | Take/skip | Medium | [Link](https://leetcode.com/problems/house-robber/) |
| Coin Change | Unbounded knapsack | Medium | [Link](https://leetcode.com/problems/coin-change/) |
| Longest Increasing Subsequence | Ending at i | Medium | [Link](https://leetcode.com/problems/longest-increasing-subsequence/) |
| Word Break | Boolean DP | Medium | [Link](https://leetcode.com/problems/word-break/) |
| Decode Ways | Counting paths | Medium | [Link](https://leetcode.com/problems/decode-ways/) |
| Maximum Subarray | Kadane's | Medium | [Link](https://leetcode.com/problems/maximum-subarray/) |
