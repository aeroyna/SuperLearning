# Two Pointers

The two pointers technique uses two indices to traverse a data structure simultaneously, reducing time complexity from O(n²) to O(n) for many problems. Understanding when and how pointers move is the key to mastering this pattern.

## Overview

Two pointers work by maintaining two positions in the data structure and moving them based on certain conditions. The technique eliminates redundant comparisons that would occur in a brute-force nested loop approach.

## Topics

- [3.3.1 Two Pointers Fundamentals](01_two_pointers.md)

## Core Patterns

### Pattern 1: Opposite Direction (Converging)

Pointers start at opposite ends and move toward each other.

**When to use**: Sorted arrays, palindrome checks, pair-sum problems

>[!example]- C++
>```cpp
>vector<int> twoSumSorted(vector<int>& nums, int target) {
>    int left = 0, right = nums.size() - 1;
>    while (left < right) {
>        int currentSum = nums[left] + nums[right];
>        if (currentSum == target) {
>            return {left, right};
>        } else if (currentSum < target) {
>            left++; // Need larger sum
>        } else {
>            right--; // Need smaller sum
>        }
>    }
>    return {};
>}
>```

>[!example]- Java
>```java
>public int[] twoSumSorted(int[] nums, int target) {
>    int left = 0, right = nums.length - 1;
>    while (left < right) {
>        int currentSum = nums[left] + nums[right];
>        if (currentSum == target) {
>            return new int[]{left, right};
>        } else if (currentSum < target) {
>            left++; // Need larger sum
>        } else {
>            right--; // Need smaller sum
>        }
>    }
>    return new int[0];
>}
>```

>[!example]- Python
>```python
>def two_sum_sorted(nums, target):
>    left, right = 0, len(nums) - 1
>    while left < right:
>        current_sum = nums[left] + nums[right]
>        if current_sum == target:
>            return [left, right]
>        elif current_sum < target:
>            left += 1  # Need larger sum
>        else:
>            right -= 1  # Need smaller sum
>    return []
>```

>[!example]- JavaScript
>```javascript
>function twoSumSorted(nums, target) {
>    let left = 0, right = nums.length - 1;
>    while (left < right) {
>        const currentSum = nums[left] + nums[right];
>        if (currentSum === target) {
>            return [left, right];
>        } else if (currentSum < target) {
>            left++; // Need larger sum
>        } else {
>            right--; // Need smaller sum
>        }
>    }
>    return [];
>}
>```

**Memory/Execution insight**: The sorted property guarantees that moving `left` right increases the sum, and moving `right` left decreases it. This monotonic property is what makes the technique work.

### Pattern 2: Same Direction (Fast/Slow)

Both pointers start at the same end, moving at different speeds or conditions.

**When to use**: Removing duplicates, partitioning, cycle detection

>[!example]- C++
>```cpp
>int removeDuplicates(vector<int>& nums) {
>    if (nums.empty()) return 0;
>    int slow = 0;
>    for (int fast = 1; fast < nums.size(); fast++) {
>        if (nums[fast] != nums[slow]) {
>            slow++;
>            nums[slow] = nums[fast];
>        }
>    }
>    return slow + 1;
>}
>```

>[!example]- Java
>```java
>public int removeDuplicates(int[] nums) {
>    if (nums.length == 0) return 0;
>    int slow = 0;
>    for (int fast = 1; fast < nums.length; fast++) {
>        if (nums[fast] != nums[slow]) {
>            slow++;
>            nums[slow] = nums[fast];
>        }
>    }
>    return slow + 1;
>}
>```

>[!example]- Python
>```python
>def remove_duplicates(nums):
>    if not nums:
>        return 0
>    slow = 0
>    for fast in range(1, len(nums)):
>        if nums[fast] != nums[slow]:
>            slow += 1
>            nums[slow] = nums[fast]
>    return slow + 1
>```

>[!example]- JavaScript
>```javascript
>function removeDuplicates(nums) {
>    if (nums.length === 0) return 0;
>    let slow = 0;
>    for (let fast = 1; fast < nums.length; fast++) {
>        if (nums[fast] !== nums[slow]) {
>            slow++;
>            nums[slow] = nums[fast];
>        }
>    }
>    return slow + 1;
>}
>```

**Memory insight**: `slow` marks the boundary of the "processed" region. Elements before `slow` are the final result; elements between `slow` and `fast` are duplicates being overwritten.

### Pattern 3: Sliding Window Variant

Two pointers defining a window that expands and contracts.

See [Sliding Window](../Sliding_Window/00_sliding_window.md) for detailed coverage.

## Decision Framework

```
Is the array sorted?
├── Yes → Consider opposite-direction pointers
│   └── Looking for pair with sum/difference? → Classic two-pointer
└── No → Consider same-direction pointers
    ├── Removing/modifying in-place? → Fast/slow pointer
    └── Subarray problems? → Sliding window
```

## Common Pitfalls

1. **Off-by-one in termination**: Use `left < right` for converging, not `left <= right` (they'd point to same element)
2. **Forgetting sorted requirement**: Opposite-direction only works on sorted arrays for sum problems
3. **Incorrect pointer movement**: Moving the wrong pointer breaks the invariant
4. **Missing edge cases**: Empty array, single element, all same elements

## Complexity Analysis

| Pattern | Time | Space | Key Insight |
|---------|------|-------|-------------|
| Converging | O(n) | O(1) | Each element visited at most once |
| Fast/Slow | O(n) | O(1) | Fast visits each element once |
| Two Arrays | O(n + m) | O(1) | Merge-like traversal |

## Key Interview Problems

| Problem | Pattern | Difficulty | LeetCode Link |
| --------- | --------- | ------------ | --- |
| Two Sum II | Converging | Easy | [Link](https://leetcode.com/problems/two-sum-ii/) |
| 3Sum | Converging + iteration | Medium | [Link](https://leetcode.com/problems/3sum/) |
| Container With Most Water | Converging | Medium | [Link](https://leetcode.com/problems/container-with-most-water/) |
| Remove Duplicates | Fast/Slow | Easy | [Link](https://leetcode.com/problems/remove-duplicates/) |
| Move Zeroes | Fast/Slow | Easy | [Link](https://leetcode.com/problems/move-zeroes/) |
| Trapping Rain Water | Converging | Hard | [Link](https://leetcode.com/problems/trapping-rain-water/) |
