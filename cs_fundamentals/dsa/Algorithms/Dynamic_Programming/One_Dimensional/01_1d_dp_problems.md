## 1D Dynamic Programming Problems

1D DP problems are those where the state can be defined by a single variable, typically an index `i`. The DP table is a 1D array, and `dp[i]` represents the solution to the subproblem for the input up to index `i`. These problems are a great introduction to the DP mindset.

### Pattern 1: Simple Recurrence (Climbing Stairs / Fibonacci)
This pattern involves a recurrence relation where the current state `dp[i]` depends on only one or two of the immediately preceding states.

#### Example: Climbing Stairs (LeetCode #70)
**Problem**: You are climbing a staircase. It takes `n` steps to reach the top. Each time you can either climb 1 or 2 steps. In how many distinct ways can you climb to the top?

- **State**: `dp[i]` = the number of distinct ways to reach step `i`.
- **Recurrence Relation**: To reach step `i`, you must have come from either step `i-1` or step `i-2`. Therefore, the number of ways to reach step `i` is the sum of the ways to reach those previous steps.
  `dp[i] = dp[i-1] + dp[i-2]`
- **Base Cases**: `dp[0] = 1` (one way to be at the start), `dp[1] = 1` (one way to take one step).

>[!example]- C++
>```cpp
>class Solution {
>public:
>    int climbStairs(int n) {
>        if (n <= 1) return 1;
>
>        vector<int> dp(n + 1);
>        dp[0] = 1;
>        dp[1] = 1;
>
>        for (int i = 2; i <= n; i++) {
>            dp[i] = dp[i-1] + dp[i-2];
>        }
>
>        return dp[n];
>    }
>};
>```

>[!example]- Java
>```java
>class Solution {
>    public int climbStairs(int n) {
>        if (n <= 1) return 1;
>
>        int[] dp = new int[n + 1];
>        dp[0] = 1;
>        dp[1] = 1;
>
>        for (int i = 2; i <= n; i++) {
>            dp[i] = dp[i-1] + dp[i-2];
>        }
>
>        return dp[n];
>    }
>}
>```

>[!example]- Python
>```python
>def climb_stairs(n):
>    if n <= 1:
>        return 1
>
>    dp = [0] * (n + 1)
>    dp[0] = 1
>    dp[1] = 1
>
>    for i in range(2, n + 1):
>        dp[i] = dp[i-1] + dp[i-2]
>
>    return dp[n]
>```

>[!example]- JavaScript
>```javascript
>function climbStairs(n) {
>    if (n <= 1) return 1;
>
>    const dp = new Array(n + 1);
>    dp[0] = 1;
>    dp[1] = 1;
>
>    for (let i = 2; i <= n; i++) {
>        dp[i] = dp[i-1] + dp[i-2];
>    }
>
>    return dp[n];
>}
>```

This is the classic Fibonacci sequence.

### Pattern 2: Choice at Each Step (House Robber)
This pattern applies when at each step `i`, you have to make a decision (e.g., to take or not to take an item), and that decision affects which subproblems you can use.

#### Example: House Robber (LeetCode #198)
**Problem**: You are a robber planning to rob houses along a street. You cannot rob two adjacent houses. Given an array `nums` representing the money in each house, find the maximum amount of money you can rob.

- **State**: `dp[i]` = the maximum money that can be robbed from the beginning up to house `i`.
- **Recurrence Relation**: At house `i`, you have two choices:
    1. **Rob house `i`**: If you rob house `i`, you cannot have robbed house `i-1`. The total money is `nums[i] + dp[i-2]` (the money from this house plus the max money from two houses ago).
    2. **Don't rob house `i`**: If you don't rob house `i`, your total is simply the max money you could have robbed up to house `i-1`, which is `dp[i-1]`.
  The optimal choice is the maximum of these two options: `dp[i] = max(dp[i-1], nums[i] + dp[i-2])`.
- **Base Cases**: `dp[0] = nums[0]`, `dp[1] = max(nums[0], nums[1])`.

>[!example]- C++
>```cpp
>class Solution {
>public:
>    int rob(vector<int>& nums) {
>        if (nums.empty()) return 0;
>        if (nums.size() <= 2) return *max_element(nums.begin(), nums.end());
>
>        vector<int> dp(nums.size());
>        dp[0] = nums[0];
>        dp[1] = max(nums[0], nums[1]);
>
>        for (int i = 2; i < nums.size(); i++) {
>            dp[i] = max(dp[i-1], nums[i] + dp[i-2]);
>        }
>
>        return dp[nums.size() - 1];
>    }
>};
>```

>[!example]- Java
>```java
>class Solution {
>    public int rob(int[] nums) {
>        if (nums.length == 0) return 0;
>        if (nums.length <= 2) {
>            int max = nums[0];
>            for (int num : nums) max = Math.max(max, num);
>            return max;
>        }
>
>        int[] dp = new int[nums.length];
>        dp[0] = nums[0];
>        dp[1] = Math.max(nums[0], nums[1]);
>
>        for (int i = 2; i < nums.length; i++) {
>            dp[i] = Math.max(dp[i-1], nums[i] + dp[i-2]);
>        }
>
>        return dp[nums.length - 1];
>    }
>}
>```

>[!example]- Python
>```python
>def rob(nums):
>    if not nums:
>        return 0
>    if len(nums) <= 2:
>        return max(nums)
>
>    dp = [0] * len(nums)
>    dp[0] = nums[0]
>    dp[1] = max(nums[0], nums[1])
>
>    for i in range(2, len(nums)):
>        dp[i] = max(dp[i-1], nums[i] + dp[i-2])
>
>    return dp[-1]
>```

>[!example]- JavaScript
>```javascript
>function rob(nums) {
>    if (nums.length === 0) return 0;
>    if (nums.length <= 2) return Math.max(...nums);
>
>    const dp = new Array(nums.length);
>    dp[0] = nums[0];
>    dp[1] = Math.max(nums[0], nums[1]);
>
>    for (let i = 2; i < nums.length; i++) {
>        dp[i] = Math.max(dp[i-1], nums[i] + dp[i-2]);
>    }
>
>    return dp[nums.length - 1];
>}
>```

Many 1D DP problems fall into one of these two patterns.
