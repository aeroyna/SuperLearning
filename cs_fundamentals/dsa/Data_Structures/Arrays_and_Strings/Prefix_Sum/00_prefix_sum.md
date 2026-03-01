# Prefix Sum

Prefix sum is a preprocessing technique that enables O(1) range sum queries after O(n) preprocessing. It transforms problems that would require O(n) per query into O(1) per query, making it essential for problems involving cumulative values.

## Overview

A prefix sum array stores cumulative sums where `prefix[i]` = sum of elements from index 0 to i-1. This allows computing any range sum as a simple subtraction: `sum(i, j) = prefix[j+1] - prefix[i]`.

## Topics

- [3.5.1 Prefix Sum Fundamentals](01_prefix_sum.md)

## Core Concept

### Building the Prefix Array

>[!example]- C++
>```cpp
>vector<int> buildPrefixSum(vector<int>& nums) {
>    int n = nums.size();
>    vector<int> prefix(n + 1, 0); // prefix[0] = 0 for convenience
>    for (int i = 0; i < n; i++) {
>        prefix[i + 1] = prefix[i] + nums[i];
>    }
>    return prefix;
>}
>```

>[!example]- Java
>```java
>public int[] buildPrefixSum(int[] nums) {
>    int n = nums.length;
>    int[] prefix = new int[n + 1]; // prefix[0] = 0 for convenience
>    for (int i = 0; i < n; i++) {
>        prefix[i + 1] = prefix[i] + nums[i];
>    }
>    return prefix;
>}
>```

>[!example]- Python
>```python
>def build_prefix_sum(nums):
>    n = len(nums)
>    prefix = [0] * (n + 1)  # prefix[0] = 0 for convenience
>    for i in range(n):
>        prefix[i + 1] = prefix[i] + nums[i]
>    return prefix
>```

>[!example]- JavaScript
>```javascript
>function buildPrefixSum(nums) {
>    const n = nums.length;
>    const prefix = new Array(n + 1).fill(0); // prefix[0] = 0 for convenience
>    for (let i = 0; i < n; i++) {
>        prefix[i + 1] = prefix[i] + nums[i];
>    }
>    return prefix;
>}
>```

**Memory layout**: For `nums = [1, 2, 3, 4]`:
```
nums:   [1,  2,  3,  4]
prefix: [0,  1,  3,  6,  10]
         ^   ^   ^   ^   ^
         |   |   |   |   sum(0,3)
         |   |   |   sum(0,2)
         |   |   sum(0,1)
         |   sum(0,0)
         base (empty sum)
```

### Range Sum Query

>[!example]- C++
>```cpp
>// Sum of nums[left:right+1]
>int rangeSum(vector<int>& prefix, int left, int right) {
>    return prefix[right + 1] - prefix[left];
>}
>```

>[!example]- Java
>```java
>// Sum of nums[left:right+1]
>public int rangeSum(int[] prefix, int left, int right) {
>    return prefix[right + 1] - prefix[left];
>}
>```

>[!example]- Python
>```python
>def range_sum(prefix, left, right):
>    # Sum of nums[left:right+1]
>    return prefix[right + 1] - prefix[left]
>```

>[!example]- JavaScript
>```javascript
>// Sum of nums[left:right+1]
>function rangeSum(prefix, left, right) {
>    return prefix[right + 1] - prefix[left];
>}
>```

**Why this works**:
- `prefix[right + 1]` = sum from 0 to right
- `prefix[left]` = sum from 0 to left-1
- Subtracting cancels everything before left

## Common Patterns

### Pattern 1: Subarray Sum Equals K

Count subarrays with exact sum using prefix sums + hash map.

>[!example]- C++
>```cpp
>int subarraySum(vector<int>& nums, int k) {
>    int count = 0;
>    int prefixSum = 0;
>    unordered_map<int, int> seen;
>    seen[0] = 1; // Empty prefix has sum 0
>
>    for (int num : nums) {
>        prefixSum += num;
>        // If (prefixSum - k) exists, there's a subarray with sum k
>        if (seen.count(prefixSum - k)) {
>            count += seen[prefixSum - k];
>        }
>        seen[prefixSum]++;
>    }
>    return count;
>}
>```

>[!example]- Java
>```java
>public int subarraySum(int[] nums, int k) {
>    int count = 0;
>    int prefixSum = 0;
>    Map<Integer, Integer> seen = new HashMap<>();
>    seen.put(0, 1); // Empty prefix has sum 0
>
>    for (int num : nums) {
>        prefixSum += num;
>        // If (prefixSum - k) exists, there's a subarray with sum k
>        if (seen.containsKey(prefixSum - k)) {
>            count += seen.get(prefixSum - k);
>        }
>        seen.put(prefixSum, seen.getOrDefault(prefixSum, 0) + 1);
>    }
>    return count;
>}
>```

>[!example]- Python
>```python
>def subarray_sum(nums, k):
>    count = 0
>    prefix_sum = 0
>    seen = {0: 1}  # Empty prefix has sum 0
>
>    for num in nums:
>        prefix_sum += num
>        # If (prefix_sum - k) exists, there's a subarray with sum k
>        if prefix_sum - k in seen:
>            count += seen[prefix_sum - k]
>        seen[prefix_sum] = seen.get(prefix_sum, 0) + 1
>
>    return count
>```

>[!example]- JavaScript
>```javascript
>function subarraySum(nums, k) {
>    let count = 0;
>    let prefixSum = 0;
>    const seen = new Map();
>    seen.set(0, 1); // Empty prefix has sum 0
>
>    for (const num of nums) {
>        prefixSum += num;
>        // If (prefixSum - k) exists, there's a subarray with sum k
>        if (seen.has(prefixSum - k)) {
>            count += seen.get(prefixSum - k);
>        }
>        seen.set(prefixSum, (seen.get(prefixSum) || 0) + 1);
>    }
>    return count;
>}
>```

**Insight**: If `prefix[j] - prefix[i] = k`, then `prefix[i] = prefix[j] - k`. We're looking for earlier prefix sums that differ from current by exactly k.

### Pattern 2: Divisibility Queries

Find subarrays divisible by k using prefix sum modulo.

```python
def subarrays_div_by_k(nums, k):
    count = 0
    prefix_mod = 0
    mod_count = {0: 1}

    for num in nums:
        prefix_mod = (prefix_mod + num % k + k) % k  # Handle negatives
        if prefix_mod in mod_count:
            count += mod_count[prefix_mod]
        mod_count[prefix_mod] = mod_count.get(prefix_mod, 0) + 1

    return count
```

**Mathematical insight**: If two prefix sums have the same remainder when divided by k, their difference is divisible by k.

### Pattern 3: 2D Prefix Sum

Extend to matrices for O(1) rectangular region sums.

```python
def build_2d_prefix(matrix):
    if not matrix:
        return []
    m, n = len(matrix), len(matrix[0])
    prefix = [[0] * (n + 1) for _ in range(m + 1)]

    for i in range(m):
        for j in range(n):
            prefix[i+1][j+1] = (matrix[i][j] + prefix[i][j+1]
                               + prefix[i+1][j] - prefix[i][j])
    return prefix

def region_sum(prefix, r1, c1, r2, c2):
    return (prefix[r2+1][c2+1] - prefix[r1][c2+1]
            - prefix[r2+1][c1] + prefix[r1][c1])
```

**Visualization** (inclusion-exclusion):
```
+-------+-------+
|   A   |   B   |
+-------+-------+
|   C   | query |
+-------+-------+

query = total - A - B - C + overlap
      = prefix[r2+1][c2+1] - prefix[r1][c2+1] - prefix[r2+1][c1] + prefix[r1][c1]
```

## Decision Framework

```
Need range sum queries?
├── Single query → Simple loop O(n)
├── Multiple queries, static array → Prefix sum
│   ├── 1D array → 1D prefix sum
│   └── 2D matrix → 2D prefix sum
└── Multiple queries, dynamic array → Segment tree or Fenwick tree
```

## Common Pitfalls

1. **Off-by-one errors**: Remember `prefix` has length `n+1`, not `n`
2. **Forgetting base case**: `prefix[0] = 0` is crucial for sums starting at index 0
3. **Integer overflow**: For large arrays, use 64-bit integers
4. **Negative numbers**: The technique handles negatives correctly

## Complexity Analysis

| Operation | Time | Space |
|-----------|------|-------|
| Build prefix (1D) | O(n) | O(n) |
| Range query (1D) | O(1) | - |
| Build prefix (2D) | O(m*n) | O(m*n) |
| Region query (2D) | O(1) | - |

## Key Interview Problems

| Problem | Variant | Difficulty | LeetCode Link |
| --------- | --------- | ------------ | --- |
| Range Sum Query - Immutable | Basic | Easy | [Link](https://leetcode.com/problems/range-sum-query-immutable/) |
| Subarray Sum Equals K | With hash map | Medium | [Link](https://leetcode.com/problems/subarray-sum-equals-k/) |
| Continuous Subarray Sum | Divisibility | Medium | [Link](https://leetcode.com/problems/continuous-subarray-sum/) |
| Range Sum Query 2D | 2D prefix | Medium | [Link](https://leetcode.com/problems/range-sum-query-2d-immutable/) |
| Product of Array Except Self | Prefix product | Medium | [Link](https://leetcode.com/problems/product-of-array-except-self/) |
