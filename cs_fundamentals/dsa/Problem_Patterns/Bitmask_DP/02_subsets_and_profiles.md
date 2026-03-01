# Bitmask DP: Subsets and Partitioning

Another common application of Bitmask DP is partitioning a set into `K` subsets with equal sum, or matching problems (like assigning tasks).

## Partition to K Equal Sum Subsets

**Problem**: Given an array of integers, determine if it can be partitioned into `k` subsets that all have equal sum.

### Approach
We fill subsets one by one.
- **State**: `dp[mask]` = current sum of the subset we are currently building.
- If `dp[mask] == -1`, this state is invalid.
- If `current_sum % target == 0`, we finished one subset and start a new one (reset sum to 0).

### Implementation

>[!example]- C++
>```cpp
>bool canPartitionKSubsets(vector<int>& nums, int k) {
>    int sum = accumulate(nums.begin(), nums.end(), 0);
>    if (sum % k != 0) return false;
>    int target = sum / k;
>    int n = nums.size();
>    int ALL_USED = (1 << n) - 1;
>    
>    // dp[mask] stores the current sum of the subset being built
>    // -1 indicates this mask is unreachable
>    vector<int> dp(1 << n, -1);
>    dp[0] = 0;
>    
>    for (int mask = 0; mask < (1 << n); mask++) {
>        if (dp[mask] == -1) continue; // Unreachable state
>        
>        for (int i = 0; i < n; i++) {
>            // If nums[i] is not yet used
>            if (!((mask >> i) & 1)) {
>                // If adding nums[i] doesn't exceed target
>                if (dp[mask] + nums[i] <= target) {
>                    int nextMask = mask | (1 << i);
>                    dp[nextMask] = (dp[mask] + nums[i]) % target;
>                }
>            }
>        }
>    }
>    return dp[ALL_USED] == 0;
>}
>```

>[!example]- Java
>```java
>public boolean canPartitionKSubsets(int[] nums, int k) {
>    int sum = Arrays.stream(nums).sum();
>    if (sum % k != 0) return false;
>    int target = sum / k;
>    int n = nums.length;
>    int ALL_USED = (1 << n) - 1;
>    
>    int[] dp = new int[1 << n];
>    Arrays.fill(dp, -1);
>    dp[0] = 0;
>    
>    for (int mask = 0; mask < (1 << n); mask++) {
>        if (dp[mask] == -1) continue;
>        
>        for (int i = 0; i < n; i++) {
>            if (((mask >> i) & 1) == 0) {
>                if (dp[mask] + nums[i] <= target) {
>                    int nextMask = mask | (1 << i);
>                    dp[nextMask] = (dp[mask] + nums[i]) % target;
>                }
>            }
>        }
>    }
>    return dp[ALL_USED] == 0;
>}
>```

>[!example]- Python
>```python
>def canPartitionKSubsets(nums, k):
>    total_sum = sum(nums)
>    if total_sum % k != 0: return False
>    target = total_sum // k
>    n = len(nums)
>    
>    # dp[mask] = current sum of subset
>    dp = [-1] * (1 << n)
>    dp[0] = 0
>    
>    for mask in range(1 << n):
>        if dp[mask] == -1: continue
>        
>        for i in range(n):
>            if not ((mask >> i) & 1):
>                if dp[mask] + nums[i] <= target:
>                    next_mask = mask | (1 << i)
>                    dp[next_mask] = (dp[mask] + nums[i]) % target
>                    
>    return dp[(1 << n) - 1] == 0
>```

>[!example]- JavaScript
>```javascript
>var canPartitionKSubsets = function(nums, k) {
>    const sum = nums.reduce((a, b) => a + b, 0);
>    if (sum % k !== 0) return false;
>    const target = sum / k;
>    const n = nums.length;
>    
>    const dp = new Array(1 << n).fill(-1);
>    dp[0] = 0;
>    
>    for (let mask = 0; mask < (1 << n); mask++) {
>        if (dp[mask] === -1) continue;
>        
>        for (let i = 0; i < n; i++) {
>            if (!((mask >> i) & 1)) {
>                if (dp[mask] + nums[i] <= target) {
>                    const nextMask = mask | (1 << i);
>                    dp[nextMask] = (dp[mask] + nums[i]) % target;
>                }
>            }
>        }
>    }
>    return dp[(1 << n) - 1] === 0;
>};
>```
