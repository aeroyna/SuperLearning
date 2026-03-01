# Prefix Sum Technique

Prefix sum is a preprocessing technique that enables O(1) range sum queries after O(n) preprocessing.

## Core Concept

Build an array `prefix` where `prefix[i]` is the sum of all elements from index 0 to i.

```
nums   = [5,  2,  1,  6,  3,  8]
prefix = [5,  7,  8, 14, 17, 25]
         ↑   ↑   ↑   ↑   ↑   ↑
        5  5+2 7+1 8+6 14+3 17+8
```

## Range Sum Query

To find sum of subarray from index `i` to `j`:

```
sum(i, j) = prefix[j] - prefix[i-1]
```

**Why this works:**
- `prefix[j]` = sum of elements 0 to j
- `prefix[i-1]` = sum of elements 0 to i-1
- Difference = sum of elements i to j

```
Array:  [5,  2,  1,  6,  3,  8]
         ----  [i=2  to  j=4]

prefix[4] = 17  (sum of indices 0-4)
prefix[1] = 7   (sum of indices 0-1)
sum(2,4)  = 17 - 7 = 10  ✓ (1 + 6 + 3 = 10)
```

## Building Prefix Sum

>[!example]- C++
>```cpp
>#include <vector>
>using namespace std;
>
>vector<int> buildPrefixSum(const vector<int>& nums) {
>    vector<int> prefix(nums.size() + 1, 0);  // Extra element for easier indexing
>    for (int i = 0; i < nums.size(); i++) {
>        prefix[i + 1] = prefix[i] + nums[i];
>    }
>    return prefix;
>}
>
>// Usage
>vector<int> nums = {5, 2, 1, 6, 3, 8};
>vector<int> prefix = buildPrefixSum(nums);
>// prefix = [0, 5, 7, 8, 14, 17, 25]
>
>// Sum from index i to j (inclusive)
>int rangeSum(const vector<int>& prefix, int i, int j) {
>    return prefix[j + 1] - prefix[i];
>}
>```

>[!example]- Java
>```java
>public int[] buildPrefixSum(int[] nums) {
>    int[] prefix = new int[nums.length + 1];  // Extra element for easier indexing
>    for (int i = 0; i < nums.length; i++) {
>        prefix[i + 1] = prefix[i] + nums[i];
>    }
>    return prefix;
>}
>
>// Usage
>int[] nums = {5, 2, 1, 6, 3, 8};
>int[] prefix = buildPrefixSum(nums);
>// prefix = [0, 5, 7, 8, 14, 17, 25]
>
>// Sum from index i to j (inclusive)
>public int rangeSum(int[] prefix, int i, int j) {
>    return prefix[j + 1] - prefix[i];
>}
>```

>[!example]- Python
>```python
>def buildPrefixSum(nums):
>    prefix = [0] * (len(nums) + 1)  # Extra element for easier indexing
>    for i in range(len(nums)):
>        prefix[i + 1] = prefix[i] + nums[i]
>    return prefix
>
># Usage
>nums = [5, 2, 1, 6, 3, 8]
>prefix = buildPrefixSum(nums)
># prefix = [0, 5, 7, 8, 14, 17, 25]
>
># Sum from index i to j (inclusive)
>def rangeSum(prefix, i, j):
>    return prefix[j + 1] - prefix[i]
>```

>[!example]- JavaScript
>```javascript
>function buildPrefixSum(nums) {
>    const prefix = new Array(nums.length + 1).fill(0);  // Extra element for easier indexing
>    for (let i = 0; i < nums.length; i++) {
>        prefix[i + 1] = prefix[i] + nums[i];
>    }
>    return prefix;
>}
>
>// Usage
>const nums = [5, 2, 1, 6, 3, 8];
>const prefix = buildPrefixSum(nums);
>// prefix = [0, 5, 7, 8, 14, 17, 25]
>
>// Sum from index i to j (inclusive)
>function rangeSum(prefix, i, j) {
>    return prefix[j + 1] - prefix[i];
>}
>```

### Without Extra Space

>[!example]- C++
>```cpp
>// In-place prefix sum
>void buildInPlace(vector<int>& nums) {
>    for (int i = 1; i < nums.size(); i++) {
>        nums[i] += nums[i - 1];
>    }
>}
>```

>[!example]- Java
>```java
>// In-place prefix sum
>public void buildInPlace(int[] nums) {
>    for (int i = 1; i < nums.length; i++) {
>        nums[i] += nums[i - 1];
>    }
>}
>```

>[!example]- Python
>```python
># In-place prefix sum
>def buildInPlace(nums):
>    for i in range(1, len(nums)):
>        nums[i] += nums[i - 1]
>```

>[!example]- JavaScript
>```javascript
>// In-place prefix sum
>function buildInPlace(nums) {
>    for (let i = 1; i < nums.length; i++) {
>        nums[i] += nums[i - 1];
>    }
>}
>```

## Common Patterns

### Pattern 1: Range Sum Query

>[!example]- C++
>```cpp
>class NumArray {
>private:
>    vector<int> prefix;
>
>public:
>    NumArray(vector<int>& nums) {
>        prefix.push_back(0);
>        for (int num : nums) {
>            prefix.push_back(prefix.back() + num);
>        }
>    }
>
>    int sumRange(int left, int right) {
>        return prefix[right + 1] - prefix[left];
>    }
>};
>```

>[!example]- Java
>```java
>class NumArray {
>    private int[] prefix;
>
>    public NumArray(int[] nums) {
>        prefix = new int[nums.length + 1];
>        for (int i = 0; i < nums.length; i++) {
>            prefix[i + 1] = prefix[i] + nums[i];
>        }
>    }
>
>    public int sumRange(int left, int right) {
>        return prefix[right + 1] - prefix[left];
>    }
>}
>```

>[!example]- Python
>```python
>class NumArray:
>    def __init__(self, nums):
>        self.prefix = [0]
>        for num in nums:
>            self.prefix.append(self.prefix[-1] + num)
>
>    def sumRange(self, left, right):
>        return self.prefix[right + 1] - self.prefix[left]
>```

>[!example]- JavaScript
>```javascript
>class NumArray {
>    constructor(nums) {
>        this.prefix = [0];
>        for (const num of nums) {
>            this.prefix.push(this.prefix[this.prefix.length - 1] + num);
>        }
>    }
>
>    sumRange(left, right) {
>        return this.prefix[right + 1] - this.prefix[left];
>    }
>}
>```

### Pattern 2: Find Subarray with Target Sum

>[!example]- C++
>```cpp
>#include <unordered_map>
>
>// Count subarrays with sum equal to k
>int subarraySum(vector<int>& nums, int k) {
>    unordered_map<int, int> prefix_count;
>    prefix_count[0] = 1;
>    int current_sum = 0;
>    int count = 0;
>
>    for (int num : nums) {
>        current_sum += num;
>
>        // If (current_sum - k) exists, we found subarrays
>        if (prefix_count.find(current_sum - k) != prefix_count.end()) {
>            count += prefix_count[current_sum - k];
>        }
>
>        prefix_count[current_sum]++;
>    }
>
>    return count;
>}
>```

>[!example]- Java
>```java
>import java.util.HashMap;
>
>// Count subarrays with sum equal to k
>public int subarraySum(int[] nums, int k) {
>    HashMap<Integer, Integer> prefixCount = new HashMap<>();
>    prefixCount.put(0, 1);
>    int currentSum = 0;
>    int count = 0;
>
>    for (int num : nums) {
>        currentSum += num;
>
>        // If (currentSum - k) exists, we found subarrays
>        if (prefixCount.containsKey(currentSum - k)) {
>            count += prefixCount.get(currentSum - k);
>        }
>
>        prefixCount.put(currentSum, prefixCount.getOrDefault(currentSum, 0) + 1);
>    }
>
>    return count;
>}
>```

>[!example]- Python
>```python
>def subarraySum(nums, k):
>    # Count subarrays with sum equal to k
>    prefix_count = {0: 1}
>    current_sum = 0
>    count = 0
>
>    for num in nums:
>        current_sum += num
>
>        # If (current_sum - k) exists, we found subarrays
>        if current_sum - k in prefix_count:
>            count += prefix_count[current_sum - k]
>
>        prefix_count[current_sum] = prefix_count.get(current_sum, 0) + 1
>
>    return count
>```

>[!example]- JavaScript
>```javascript
>// Count subarrays with sum equal to k
>function subarraySum(nums, k) {
>    const prefixCount = new Map();
>    prefixCount.set(0, 1);
>    let currentSum = 0;
>    let count = 0;
>
>    for (const num of nums) {
>        currentSum += num;
>
>        // If (currentSum - k) exists, we found subarrays
>        if (prefixCount.has(currentSum - k)) {
>            count += prefixCount.get(currentSum - k);
>        }
>
>        prefixCount.set(currentSum, (prefixCount.get(currentSum) || 0) + 1);
>    }
>
>    return count;
>}
>```

### Pattern 3: Split Array

>[!example]- C++
>```cpp
>int waysToSplitArray(vector<int>& nums) {
>    long long total = 0;
>    for (int num : nums) total += num;
>
>    long long left_sum = 0;
>    int count = 0;
>
>    for (int i = 0; i < nums.size() - 1; i++) {
>        left_sum += nums[i];
>        long long right_sum = total - left_sum;
>        if (left_sum >= right_sum) {
>            count++;
>        }
>    }
>
>    return count;
>}
>```

>[!example]- Java
>```java
>public int waysToSplitArray(int[] nums) {
>    long total = 0;
>    for (int num : nums) total += num;
>
>    long leftSum = 0;
>    int count = 0;
>
>    for (int i = 0; i < nums.length - 1; i++) {
>        leftSum += nums[i];
>        long rightSum = total - leftSum;
>        if (leftSum >= rightSum) {
>            count++;
>        }
>    }
>
>    return count;
>}
>```

>[!example]- Python
>```python
>def waysToSplitArray(nums):
>    total = sum(nums)
>    left_sum = 0
>    count = 0
>
>    for i in range(len(nums) - 1):
>        left_sum += nums[i]
>        right_sum = total - left_sum
>        if left_sum >= right_sum:
>            count += 1
>
>    return count
>```

>[!example]- JavaScript
>```javascript
>function waysToSplitArray(nums) {
>    const total = nums.reduce((sum, num) => sum + num, 0);
>    let leftSum = 0;
>    let count = 0;
>
>    for (let i = 0; i < nums.length - 1; i++) {
>        leftSum += nums[i];
>        const rightSum = total - leftSum;
>        if (leftSum >= rightSum) {
>            count++;
>        }
>    }
>
>    return count;
>}
>```

### Pattern 4: Equilibrium Index

Find index where left sum equals right sum.

>[!example]- C++
>```cpp
>int findPivotIndex(vector<int>& nums) {
>    int total = 0;
>    for (int num : nums) total += num;
>
>    int left_sum = 0;
>    for (int i = 0; i < nums.size(); i++) {
>        // Right sum = total - left_sum - current element
>        if (left_sum == total - left_sum - nums[i]) {
>            return i;
>        }
>        left_sum += nums[i];
>    }
>
>    return -1;
>}
>```

>[!example]- Java
>```java
>public int findPivotIndex(int[] nums) {
>    int total = 0;
>    for (int num : nums) total += num;
>
>    int leftSum = 0;
>    for (int i = 0; i < nums.length; i++) {
>        // Right sum = total - leftSum - current element
>        if (leftSum == total - leftSum - nums[i]) {
>            return i;
>        }
>        leftSum += nums[i];
>    }
>
>    return -1;
>}
>```

>[!example]- Python
>```python
>def findPivotIndex(nums):
>    total = sum(nums)
>    left_sum = 0
>
>    for i, num in enumerate(nums):
>        # Right sum = total - left_sum - current element
>        if left_sum == total - left_sum - num:
>            return i
>        left_sum += num
>
>    return -1
>```

>[!example]- JavaScript
>```javascript
>function findPivotIndex(nums) {
>    const total = nums.reduce((sum, num) => sum + num, 0);
>    let leftSum = 0;
>
>    for (let i = 0; i < nums.length; i++) {
>        // Right sum = total - leftSum - current element
>        if (leftSum === total - leftSum - nums[i]) {
>            return i;
>        }
>        leftSum += nums[i];
>    }
>
>    return -1;
>}
>```

## 2D Prefix Sum

For matrix queries, extend to 2D.

>[!example]- C++
>```cpp
>#include <vector>
>using namespace std;
>
>vector<vector<int>> build2DPrefixSum(const vector<vector<int>>& matrix) {
>    if (matrix.empty()) return {};
>
>    int rows = matrix.size(), cols = matrix[0].size();
>    vector<vector<int>> prefix(rows + 1, vector<int>(cols + 1, 0));
>
>    for (int i = 0; i < rows; i++) {
>        for (int j = 0; j < cols; j++) {
>            prefix[i + 1][j + 1] = matrix[i][j]
>                                 + prefix[i][j + 1]
>                                 + prefix[i + 1][j]
>                                 - prefix[i][j];
>        }
>    }
>
>    return prefix;
>}
>
>int queryRegion(const vector<vector<int>>& prefix, int r1, int c1, int r2, int c2) {
>    return prefix[r2 + 1][c2 + 1]
>         - prefix[r1][c2 + 1]
>         - prefix[r2 + 1][c1]
>         + prefix[r1][c1];
>}
>```

>[!example]- Java
>```java
>public int[][] build2DPrefixSum(int[][] matrix) {
>    if (matrix.length == 0) return new int[0][0];
>
>    int rows = matrix.length, cols = matrix[0].length;
>    int[][] prefix = new int[rows + 1][cols + 1];
>
>    for (int i = 0; i < rows; i++) {
>        for (int j = 0; j < cols; j++) {
>            prefix[i + 1][j + 1] = matrix[i][j]
>                                 + prefix[i][j + 1]
>                                 + prefix[i + 1][j]
>                                 - prefix[i][j];
>        }
>    }
>
>    return prefix;
>}
>
>public int queryRegion(int[][] prefix, int r1, int c1, int r2, int c2) {
>    return prefix[r2 + 1][c2 + 1]
>         - prefix[r1][c2 + 1]
>         - prefix[r2 + 1][c1]
>         + prefix[r1][c1];
>}
>```

>[!example]- Python
>```python
>def build2DPrefixSum(matrix):
>    if not matrix:
>        return []
>
>    rows, cols = len(matrix), len(matrix[0])
>    prefix = [[0] * (cols + 1) for _ in range(rows + 1)]
>
>    for i in range(rows):
>        for j in range(cols):
>            prefix[i + 1][j + 1] = (matrix[i][j]
>                                   + prefix[i][j + 1]
>                                   + prefix[i + 1][j]
>                                   - prefix[i][j])
>
>    return prefix
>
>def queryRegion(prefix, r1, c1, r2, c2):
>    return (prefix[r2 + 1][c2 + 1]
>            - prefix[r1][c2 + 1]
>            - prefix[r2 + 1][c1]
>            + prefix[r1][c1])
>```

>[!example]- JavaScript
>```javascript
>function build2DPrefixSum(matrix) {
>    if (matrix.length === 0) return [];
>
>    const rows = matrix.length, cols = matrix[0].length;
>    const prefix = Array.from({length: rows + 1},
>                             () => Array(cols + 1).fill(0));
>
>    for (let i = 0; i < rows; i++) {
>        for (let j = 0; j < cols; j++) {
>            prefix[i + 1][j + 1] = matrix[i][j]
>                                 + prefix[i][j + 1]
>                                 + prefix[i + 1][j]
>                                 - prefix[i][j];
>        }
>    }
>
>    return prefix;
>}
>
>function queryRegion(prefix, r1, c1, r2, c2) {
>    return prefix[r2 + 1][c2 + 1]
>         - prefix[r1][c2 + 1]
>         - prefix[r2 + 1][c1]
>         + prefix[r1][c1];
>}
>```

## Prefix Sum Variants

### Prefix Product

>[!example]- C++
>```cpp
>vector<int> productExceptSelf(vector<int>& nums) {
>    int n = nums.size();
>    vector<int> result(n, 1);
>
>    // Prefix product (left to right)
>    int prefix = 1;
>    for (int i = 0; i < n; i++) {
>        result[i] = prefix;
>        prefix *= nums[i];
>    }
>
>    // Suffix product (right to left)
>    int suffix = 1;
>    for (int i = n - 1; i >= 0; i--) {
>        result[i] *= suffix;
>        suffix *= nums[i];
>    }
>
>    return result;
>}
>```

>[!example]- Java
>```java
>public int[] productExceptSelf(int[] nums) {
>    int n = nums.length;
>    int[] result = new int[n];
>    Arrays.fill(result, 1);
>
>    // Prefix product (left to right)
>    int prefix = 1;
>    for (int i = 0; i < n; i++) {
>        result[i] = prefix;
>        prefix *= nums[i];
>    }
>
>    // Suffix product (right to left)
>    int suffix = 1;
>    for (int i = n - 1; i >= 0; i--) {
>        result[i] *= suffix;
>        suffix *= nums[i];
>    }
>
>    return result;
>}
>```

>[!example]- Python
>```python
>def productExceptSelf(nums):
>    n = len(nums)
>    result = [1] * n
>
>    # Prefix product (left to right)
>    prefix = 1
>    for i in range(n):
>        result[i] = prefix
>        prefix *= nums[i]
>
>    # Suffix product (right to left)
>    suffix = 1
>    for i in range(n - 1, -1, -1):
>        result[i] *= suffix
>        suffix *= nums[i]
>
>    return result
>```

>[!example]- JavaScript
>```javascript
>function productExceptSelf(nums) {
>    const n = nums.length;
>    const result = new Array(n).fill(1);
>
>    // Prefix product (left to right)
>    let prefix = 1;
>    for (let i = 0; i < n; i++) {
>        result[i] = prefix;
>        prefix *= nums[i];
>    }
>
>    // Suffix product (right to left)
>    let suffix = 1;
>    for (let i = n - 1; i >= 0; i--) {
>        result[i] *= suffix;
>        suffix *= nums[i];
>    }
>
>    return result;
>}
>```

### Prefix XOR

>[!example]- C++
>```cpp
>vector<int> xorQueries(vector<int>& arr, vector<vector<int>>& queries) {
>    // Build prefix XOR
>    vector<int> prefix = {0};
>    for (int num : arr) {
>        prefix.push_back(prefix.back() ^ num);
>    }
>
>    // Answer queries
>    vector<int> result;
>    for (const auto& q : queries) {
>        result.push_back(prefix[q[1] + 1] ^ prefix[q[0]]);
>    }
>    return result;
>}
>```

>[!example]- Java
>```java
>public int[] xorQueries(int[] arr, int[][] queries) {
>    // Build prefix XOR
>    int[] prefix = new int[arr.length + 1];
>    for (int i = 0; i < arr.length; i++) {
>        prefix[i + 1] = prefix[i] ^ arr[i];
>    }
>
>    // Answer queries
>    int[] result = new int[queries.length];
>    for (int i = 0; i < queries.length; i++) {
>        result[i] = prefix[queries[i][1] + 1] ^ prefix[queries[i][0]];
>    }
>    return result;
>}
>```

>[!example]- Python
>```python
>def xorQueries(arr, queries):
>    # Build prefix XOR
>    prefix = [0]
>    for num in arr:
>        prefix.append(prefix[-1] ^ num)
>
>    # Answer queries
>    return [prefix[r + 1] ^ prefix[l] for l, r in queries]
>```

>[!example]- JavaScript
>```javascript
>function xorQueries(arr, queries) {
>    // Build prefix XOR
>    const prefix = [0];
>    for (const num of arr) {
>        prefix.push(prefix[prefix.length - 1] ^ num);
>    }
>
>    // Answer queries
>    return queries.map(([l, r]) => prefix[r + 1] ^ prefix[l]);
>}
>```

### Prefix Count

Count occurrences up to each index.

>[!example]- C++
>```cpp
>vector<int> countOccurrences(const vector<int>& arr, int target) {
>    vector<int> prefix_count(arr.size() + 1, 0);
>    for (int i = 0; i < arr.size(); i++) {
>        prefix_count[i + 1] = prefix_count[i] + (arr[i] == target ? 1 : 0);
>    }
>    return prefix_count;
>}
>
>// Count in range [i, j]
>int countInRange(const vector<int>& prefix_count, int i, int j) {
>    return prefix_count[j + 1] - prefix_count[i];
>}
>```

>[!example]- Java
>```java
>public int[] countOccurrences(int[] arr, int target) {
>    int[] prefixCount = new int[arr.length + 1];
>    for (int i = 0; i < arr.length; i++) {
>        prefixCount[i + 1] = prefixCount[i] + (arr[i] == target ? 1 : 0);
>    }
>    return prefixCount;
>}
>
>// Count in range [i, j]
>public int countInRange(int[] prefixCount, int i, int j) {
>    return prefixCount[j + 1] - prefixCount[i];
>}
>```

>[!example]- Python
>```python
>def countOccurrences(arr, target):
>    prefix_count = [0] * (len(arr) + 1)
>    for i, num in enumerate(arr):
>        prefix_count[i + 1] = prefix_count[i] + (1 if num == target else 0)
>
>    # Count in range [i, j]
>    def countInRange(i, j):
>        return prefix_count[j + 1] - prefix_count[i]
>
>    return prefix_count, countInRange
>```

>[!example]- JavaScript
>```javascript
>function countOccurrences(arr, target) {
>    const prefixCount = new Array(arr.length + 1).fill(0);
>    for (let i = 0; i < arr.length; i++) {
>        prefixCount[i + 1] = prefixCount[i] + (arr[i] === target ? 1 : 0);
>    }
>    return prefixCount;
>}
>
>// Count in range [i, j]
>function countInRange(prefixCount, i, j) {
>    return prefixCount[j + 1] - prefixCount[i];
>}
>```

## Complexity Analysis

| Operation | Time | Space |
|-----------|------|-------|
| Build prefix sum | O(n) | O(n) |
| Range query | O(1) | - |
| Build 2D prefix | O(n*m) | O(n*m) |
| 2D range query | O(1) | - |

## Practice Problems

| Problem | Type | Difficulty |
|---------|------|------------|
| Running Sum of 1d Array | Basic | Easy |
| Range Sum Query | Basic | Easy |
| Subarray Sum Equals K | Hash Map | Medium |
| Find Pivot Index | Comparison | Easy |
| Product of Array Except Self | Product | Medium |
| Range Sum Query 2D | 2D Matrix | Medium |
| Continuous Subarray Sum | Hash Map | Medium |
