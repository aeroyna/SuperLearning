# Cyclic Sort

Cyclic Sort is an efficient in-place sorting algorithm used for problems involving arrays containing numbers in a given range (usually `1` to `n` or `0` to `n`). It achieves **O(n)** time complexity with **O(1)** extra space, making it superior to standard sorting algorithms for these specific constraints.

## Concept

The core idea is to place each number at its correct index.
- If the numbers are `1` to `n`, the value `x` should be at index `x-1`.
- If the numbers are `0` to `n`, the value `x` should be at index `x`.

We iterate through the array, and if the current number is not at its correct index (and the target index doesn't already have the correct number), we swap it to its correct position.

## Algorithm

>[!example]- C++
>```cpp
>void cyclicSort(vector<int>& nums) {
>    int i = 0;
>    while (i < nums.size()) {
>        // For 1 to n range, correct index is value - 1
>        int correctIdx = nums[i] - 1;
>        if (nums[i] != nums[correctIdx]) {
>            swap(nums[i], nums[correctIdx]);
>        } else {
>            i++;
>        }
>    }
>}
>```

>[!example]- Java
>```java
>void cyclicSort(int[] nums) {
>    int i = 0;
>    while (i < nums.length) {
>        // For 1 to n range, correct index is value - 1
>        int correctIdx = nums[i] - 1;
>        if (nums[i] != nums[correctIdx]) {
>            int temp = nums[i];
>            nums[i] = nums[correctIdx];
>            nums[correctIdx] = temp;
>        } else {
>            i++;
>        }
>    }
>}
>```

>[!example]- Python
>```python
>def cyclic_sort(nums):
>    i = 0
>    while i < len(nums):
>        # For 1 to n range, correct index is value - 1
>        correct_idx = nums[i] - 1
>        if nums[i] != nums[correct_idx]:
>            nums[i], nums[correct_idx] = nums[correct_idx], nums[i]
>        else:
>            i += 1
>```

>[!example]- JavaScript
>```javascript
>function cyclicSort(nums) {
>    let i = 0;
>    while (i < nums.length) {
>        // For 1 to n range, correct index is value - 1
>        const correctIdx = nums[i] - 1;
>        if (nums[i] !== nums[correctIdx]) {
>            [nums[i], nums[correctIdx]] = [nums[correctIdx], nums[i]];
>        } else {
>            i++;
>        }
>    }
>}
>```

## Pattern Recognition

**Signals**:
- "Array containing numbers from 1 to n"
- "Find the missing number in range 0 to n"
- "Find the duplicate number in range 1 to n"
- Constraint: O(n) time and O(1) space

## Common Problems

### 1. Missing Number
Given an array containing `n` distinct numbers taken from `0, 1, 2, ..., n`, find the one that is missing.
- **Approach**: Cyclic sort to place `i` at index `i`. The index where `nums[i] != i` is the missing number.

### 2. Find All Numbers Disappeared in an Array
Given an array of integers where `1 ≤ a[i] ≤ n` (some elements appear twice and others appear once), find all elements of `[1, n]` inclusive that do not appear in this array.
- **Approach**: Cyclic sort. Indices that don't satisfy `nums[i] == i + 1` are the missing numbers.

### 3. Find the Duplicate Number
Given an array of integers containing `n + 1` integers where each integer is in the range `[1, n]` inclusive. There is only one repeated number.
- **Approach**: Cyclic sort. If we try to place a number `x` at `x-1` and `nums[x-1]` is already `x`, then `x` is the duplicate.

### 4. First Missing Positive
Given an unsorted integer array, find the smallest missing positive integer.
- **Approach**: Ignore negatives and numbers > n. Place positive `x` at `x-1`. The first index `i` where `nums[i] != i + 1` implies `i + 1` is the answer.

## Complexity Analysis

- **Time Complexity**: **O(n)**. Each number is swapped at most once to its correct position. In the worst case, we do `n` swaps and `n` checks.
- **Space Complexity**: **O(1)**. In-place modification.


### Practice
- [Practice Problems](Practice_Problems/00_practice_problems.md)