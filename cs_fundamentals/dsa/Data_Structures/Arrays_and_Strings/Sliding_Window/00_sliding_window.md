# Sliding Window

The sliding window technique maintains a contiguous subarray/substring and efficiently computes properties as the window expands or contracts. It transforms O(n²) or O(n³) brute-force solutions into O(n) algorithms.

## Overview

A window is defined by two pointers: `left` (start) and `right` (end). The technique works by:
1. Expanding the window (move `right`) to include more elements
2. Contracting the window (move `left`) when constraints are violated
3. Tracking the answer as the window changes

## Topics

- [3.4.1 Sliding Window Fundamentals](01_sliding_window.md)

## Core Patterns

### Pattern 1: Fixed-Size Window

Window size is predetermined. Slide the window one position at a time.

**When to use**: Maximum/average of all subarrays of size k

>[!example]- C++
>```cpp
>int maxSumSubarrayK(vector<int>& nums, int k) {
>    if (nums.size() < k) return 0;
>
>    // Initial window
>    int windowSum = 0;
>    for (int i = 0; i < k; i++) windowSum += nums[i];
>    int maxSum = windowSum;
>
>    // Slide: remove leftmost, add rightmost
>    for (int i = k; i < nums.size(); i++) {
>        windowSum += nums[i] - nums[i - k];
>        maxSum = max(maxSum, windowSum);
>    }
>    return maxSum;
>}
>```

>[!example]- Java
>```java
>public int maxSumSubarrayK(int[] nums, int k) {
>    if (nums.length < k) return 0;
>
>    // Initial window
>    int windowSum = 0;
>    for (int i = 0; i < k; i++) windowSum += nums[i];
>    int maxSum = windowSum;
>
>    // Slide: remove leftmost, add rightmost
>    for (int i = k; i < nums.length; i++) {
>        windowSum += nums[i] - nums[i - k];
>        maxSum = Math.max(maxSum, windowSum);
>    }
>    return maxSum;
>}
>```

>[!example]- Python
>```python
>def max_sum_subarray_k(nums, k):
>    if len(nums) < k:
>        return 0
>
>    # Initial window
>    window_sum = sum(nums[:k])
>    max_sum = window_sum
>
>    # Slide: remove leftmost, add rightmost
>    for i in range(k, len(nums)):
>        window_sum += nums[i] - nums[i - k]
>        max_sum = max(max_sum, window_sum)
>
>    return max_sum
>```

>[!example]- JavaScript
>```javascript
>function maxSumSubarrayK(nums, k) {
>    if (nums.length < k) return 0;
>
>    // Initial window
>    let windowSum = 0;
>    for (let i = 0; i < k; i++) windowSum += nums[i];
>    let maxSum = windowSum;
>
>    // Slide: remove leftmost, add rightmost
>    for (let i = k; i < nums.length; i++) {
>        windowSum += nums[i] - nums[i - k];
>        maxSum = Math.max(maxSum, windowSum);
>    }
>    return maxSum;
>}
>```

**Execution insight**: Instead of recalculating the sum for each window (O(k) each), we do incremental updates (O(1) each). Total: O(n) instead of O(n*k).

### Pattern 2: Variable-Size Window (Longest)

Find the longest window satisfying a condition. Expand right, contract left only when constraint violated.

**When to use**: Longest substring with at most K distinct characters

>[!example]- C++
>```cpp
>int longestSubstringKDistinct(string s, int k) {
>    unordered_map<char, int> charCount;
>    int left = 0, maxLength = 0;
>
>    for (int right = 0; right < s.length(); right++) {
>        charCount[s[right]]++;
>
>        while (charCount.size() > k) {
>            charCount[s[left]]--;
>            if (charCount[s[left]] == 0) {
>                charCount.erase(s[left]);
>            }
>            left++;
>        }
>        maxLength = max(maxLength, right - left + 1);
>    }
>    return maxLength;
>}
>```

>[!example]- Java
>```java
>public int longestSubstringKDistinct(String s, int k) {
>    Map<Character, Integer> charCount = new HashMap<>();
>    int left = 0, maxLength = 0;
>
>    for (int right = 0; right < s.length(); right++) {
>        char c = s.charAt(right);
>        charCount.put(c, charCount.getOrDefault(c, 0) + 1);
>
>        while (charCount.size() > k) {
>            char leftChar = s.charAt(left);
>            charCount.put(leftChar, charCount.get(leftChar) - 1);
>            if (charCount.get(leftChar) == 0) {
>                charCount.remove(leftChar);
>            }
>            left++;
>        }
>        maxLength = Math.max(maxLength, right - left + 1);
>    }
>    return maxLength;
>}
>```

>[!example]- Python
>```python
>def longest_substring_k_distinct(s, k):
>    char_count = {}
>    left = 0
>    max_length = 0
>
>    for right in range(len(s)):
>        # Expand: add right character
>        char_count[s[right]] = char_count.get(s[right], 0) + 1
>
>        # Contract: remove left characters until valid
>        while len(char_count) > k:
>            char_count[s[left]] -= 1
>            if char_count[s[left]] == 0:
>                del char_count[s[left]]
>            left += 1
>
>        # Update answer (window is always valid here)
>        max_length = max(max_length, right - left + 1)
>
>    return max_length
>```

>[!example]- JavaScript
>```javascript
>function longestSubstringKDistinct(s, k) {
>    const charCount = new Map();
>    let left = 0, maxLength = 0;
>
>    for (let right = 0; right < s.length; right++) {
>        const c = s[right];
>        charCount.set(c, (charCount.get(c) || 0) + 1);
>
>        while (charCount.size > k) {
>            const leftChar = s[left];
>            charCount.set(leftChar, charCount.get(leftChar) - 1);
>            if (charCount.get(leftChar) === 0) {
>                charCount.delete(leftChar);
>            }
>            left++;
>        }
>        maxLength = Math.max(maxLength, right - left + 1);
>    }
>    return maxLength;
>}
>```

**Memory insight**: The hash map maintains the window state. Its size is bounded by the constraint (k distinct), keeping space complexity manageable.

### Pattern 3: Variable-Size Window (Shortest)

Find the shortest window satisfying a condition. Expand until valid, then contract while maintaining validity.

**When to use**: Minimum window substring, smallest subarray with sum >= target

>[!example]- C++
>```cpp
>int minSubarraySum(vector<int>& nums, int target) {
>    int left = 0, currentSum = 0;
>    int minLength = INT_MAX;
>
>    for (int right = 0; right < nums.size(); right++) {
>        currentSum += nums[right];
>
>        // Contract while valid to find minimum
>        while (currentSum >= target) {
>            minLength = min(minLength, right - left + 1);
>            currentSum -= nums[left];
>            left++;
>        }
>    }
>    return minLength == INT_MAX ? 0 : minLength;
>}
>```

>[!example]- Java
>```java
>public int minSubarraySum(int[] nums, int target) {
>    int left = 0, currentSum = 0;
>    int minLength = Integer.MAX_VALUE;
>
>    for (int right = 0; right < nums.length; right++) {
>        currentSum += nums[right];
>
>        // Contract while valid to find minimum
>        while (currentSum >= target) {
>            minLength = Math.min(minLength, right - left + 1);
>            currentSum -= nums[left];
>            left++;
>        }
>    }
>    return minLength == Integer.MAX_VALUE ? 0 : minLength;
>}
>```

>[!example]- Python
>```python
>def min_subarray_sum(nums, target):
>    left = 0
>    current_sum = 0
>    min_length = float('inf')
>
>    for right in range(len(nums)):
>        current_sum += nums[right]
>
>        # Contract while valid to find minimum
>        while current_sum >= target:
>            min_length = min(min_length, right - left + 1)
>            current_sum -= nums[left]
>            left += 1
>
>    return min_length if min_length != float('inf') else 0
>```

>[!example]- JavaScript
>```javascript
>function minSubarraySum(nums, target) {
>    let left = 0, currentSum = 0;
>    let minLength = Infinity;
>
>    for (let right = 0; right < nums.length; right++) {
>        currentSum += nums[right];
>
>        // Contract while valid to find minimum
>        while (currentSum >= target) {
>            minLength = Math.min(minLength, right - left + 1);
>            currentSum -= nums[left];
>            left++;
>        }
>    }
>    return minLength === Infinity ? 0 : minLength;
>}
>```

## Decision Framework

```
Is window size fixed?
├── Yes → Fixed-size sliding window
│   └── Use: window_sum += new - old
└── No → Variable-size window
    ├── Finding longest? → Expand freely, contract when invalid
    └── Finding shortest? → Expand until valid, contract while valid
```

## State Tracking Techniques

| State to Track | Data Structure | Example |
|----------------|---------------|---------|
| Sum | Single variable | Subarray sum |
| Character frequency | Hash map | Anagram matching |
| Distinct count | Hash map size | K distinct chars |
| Max/Min in window | Monotonic deque | Sliding window maximum |

## Common Pitfalls

1. **Wrong contraction condition**: Longest uses `while invalid`, shortest uses `while valid`
2. **Off-by-one in window size**: `right - left + 1` is the window size
3. **Forgetting to update answer**: For longest, update after contraction; for shortest, update during contraction
4. **Not handling empty result**: Check if answer was ever updated

## Complexity Analysis

| Type | Time | Space | Key Insight |
|------|------|-------|-------------|
| Fixed | O(n) | O(1) | Each element added/removed once |
| Variable | O(n) | O(k) | Left pointer only moves right, never backtracks |

## Key Interview Problems

| Problem | Type | Difficulty | LeetCode Link |
| --------- | ------ | ------------ | --- |
| Maximum Average Subarray I | Fixed | Easy | [Link](https://leetcode.com/problems/maximum-average-subarray-i/) |
| Longest Substring Without Repeating | Variable (longest) | Medium | [Link](https://leetcode.com/problems/longest-substring-without-repeating-characters/) |
| Minimum Window Substring | Variable (shortest) | Hard | [Link](https://leetcode.com/problems/minimum-window-substring/) |
| Permutation in String | Fixed with frequency | Medium | [Link](https://leetcode.com/problems/permutation-in-string/) |
| Sliding Window Maximum | Fixed with monotonic deque | Hard | [Link](https://leetcode.com/problems/sliding-window-maximum/) |
