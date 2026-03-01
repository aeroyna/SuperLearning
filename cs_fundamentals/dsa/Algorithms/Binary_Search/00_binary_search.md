# Binary Search

Binary search is an efficient algorithm for finding elements in sorted data, achieving O(log n) time complexity by halving the search space at each step.

## Overview

Binary search works on sorted arrays by repeatedly dividing the search space in half.

## Topics

- [14.1 Binary Search on Arrays](01_binary_search_arrays.md)
- [14.2 Binary Search on Solution Space](02_binary_search_solution_space.md)
- [14.3 Binary Search Templates](03_binary_search_templates.md)
- [14.4 Practice Problems](Practice_Problems/00_practice_problems.md)

## Basic Template

>[!example]- C++
>```cpp
>int binarySearch(vector<int>& arr, int target) {
>    int left = 0, right = arr.size() - 1;
>    while (left <= right) {
>        int mid = left + (right - left) / 2;
>        if (arr[mid] == target) return mid;
>        if (arr[mid] < target) left = mid + 1;
>        else right = mid - 1;
>    }
>    return -1;
>}
>```

>[!example]- Java
>```java
>public int binarySearch(int[] arr, int target) {
>    int left = 0, right = arr.length - 1;
>    while (left <= right) {
>        int mid = left + (right - left) / 2;
>        if (arr[mid] == target) return mid;
>        if (arr[mid] < target) left = mid + 1;
>        else right = mid - 1;
>    }
>    return -1;
>}
>```

>[!example]- Python
>```python
>def binarySearch(arr, target):
>    left, right = 0, len(arr) - 1
>
>    while left <= right:
>        mid = left + (right - left) // 2
>
>        if arr[mid] == target:
>            return mid
>        elif arr[mid] < target:
>            left = mid + 1
>        else:
>            right = mid - 1
>
>    return -1  # Not found
>```

>[!example]- JavaScript
>```javascript
>function binarySearch(arr, target) {
>    let left = 0, right = arr.length - 1;
>    while (left <= right) {
>        const mid = Math.floor(left + (right - left) / 2);
>        if (arr[mid] === target) return mid;
>        if (arr[mid] < target) left = mid + 1;
>        else right = mid - 1;
>    }
>    return -1;
>}
>```

## Finding Boundaries

### Find Leftmost (First) Occurrence

>[!example]- C++
>```cpp
>int findFirst(vector<int>& arr, int target) {
>    int left = 0, right = arr.size() - 1;
>    int result = -1;
>    while (left <= right) {
>        int mid = left + (right - left) / 2;
>        if (arr[mid] == target) {
>            result = mid;
>            right = mid - 1; // Keep searching left
>        } else if (arr[mid] < target) {
>            left = mid + 1;
>        } else {
>            right = mid - 1;
>        }
>    }
>    return result;
>}
>```

>[!example]- Java
>```java
>public int findFirst(int[] arr, int target) {
>    int left = 0, right = arr.length - 1;
>    int result = -1;
>    while (left <= right) {
>        int mid = left + (right - left) / 2;
>        if (arr[mid] == target) {
>            result = mid;
>            right = mid - 1; // Keep searching left
>        } else if (arr[mid] < target) {
>            left = mid + 1;
>        } else {
>            right = mid - 1;
>        }
>    }
>    return result;
>}
>```

>[!example]- Python
>```python
>def findFirst(arr, target):
>    left, right = 0, len(arr) - 1
>    result = -1
>
>    while left <= right:
>        mid = left + (right - left) // 2
>
>        if arr[mid] == target:
>            result = mid
>            right = mid - 1  # Keep searching left
>        elif arr[mid] < target:
>            left = mid + 1
>        else:
>            right = mid - 1
>
>    return result
>```

>[!example]- JavaScript
>```javascript
>function findFirst(arr, target) {
>    let left = 0, right = arr.length - 1;
>    let result = -1;
>    while (left <= right) {
>        const mid = Math.floor(left + (right - left) / 2);
>        if (arr[mid] === target) {
>            result = mid;
>            right = mid - 1; // Keep searching left
>        } else if (arr[mid] < target) {
>            left = mid + 1;
>        } else {
>            right = mid - 1;
>        }
>    }
>    return result;
>}
>```

### Find Rightmost (Last) Occurrence

>[!example]- C++
>```cpp
>int findLast(vector<int>& arr, int target) {
>    int left = 0, right = arr.size() - 1;
>    int result = -1;
>    while (left <= right) {
>        int mid = left + (right - left) / 2;
>        if (arr[mid] == target) {
>            result = mid;
>            left = mid + 1; // Keep searching right
>        } else if (arr[mid] < target) {
>            left = mid + 1;
>        } else {
>            right = mid - 1;
>        }
>    }
>    return result;
>}
>```

>[!example]- Java
>```java
>public int findLast(int[] arr, int target) {
>    int left = 0, right = arr.length - 1;
>    int result = -1;
>    while (left <= right) {
>        int mid = left + (right - left) / 2;
>        if (arr[mid] == target) {
>            result = mid;
>            left = mid + 1; // Keep searching right
>        } else if (arr[mid] < target) {
>            left = mid + 1;
>        } else {
>            right = mid - 1;
>        }
>    }
>    return result;
>}
>```

>[!example]- Python
>```python
>def findLast(arr, target):
>    left, right = 0, len(arr) - 1
>    result = -1
>
>    while left <= right:
>        mid = left + (right - left) // 2
>
>        if arr[mid] == target:
>            result = mid
>            left = mid + 1  # Keep searching right
>        elif arr[mid] < target:
>            left = mid + 1
>        else:
>            right = mid - 1
>
>    return result
>```

>[!example]- JavaScript
>```javascript
>function findLast(arr, target) {
>    let left = 0, right = arr.length - 1;
>    let result = -1;
>    while (left <= right) {
>        const mid = Math.floor(left + (right - left) / 2);
>        if (arr[mid] === target) {
>            result = mid;
>            left = mid + 1; // Keep searching right
>        } else if (arr[mid] < target) {
>            left = mid + 1;
>        } else {
>            right = mid - 1;
>        }
>    }
>    return result;
>}
>```

## Binary Search on Solution Space

When the answer itself can be binary searched:

>[!example]- C++
>```cpp
>// Find minimum value where check(x) is True
>int binarySearchOnAnswer(function<bool(int)> check, int low, int high) {
>    while (low < high) {
>        int mid = low + (high - low) / 2;
>        if (check(mid)) {
>            high = mid; // Answer could be mid or smaller
>        } else {
>            low = mid + 1; // Need larger value
>        }
>    }
>    return low;
>}
>```

>[!example]- Java
>```java
>// Find minimum value where check(x) is True
>public int binarySearchOnAnswer(Predicate<Integer> check, int low, int high) {
>    while (low < high) {
>        int mid = low + (high - low) / 2;
>        if (check.test(mid)) {
>            high = mid; // Answer could be mid or smaller
>        } else {
>            low = mid + 1; // Need larger value
>        }
>    }
>    return low;
>}
>```

>[!example]- Python
>```python
>def binarySearchOnAnswer(check, low, high):
>    """Find minimum value where check(x) is True"""
>    while low < high:
>        mid = low + (high - low) // 2
>
>        if check(mid):
>            high = mid  # Answer could be mid or smaller
>        else:
>            low = mid + 1  # Need larger value
>
>    return low
>```

>[!example]- JavaScript
>```javascript
>// Find minimum value where check(x) is True
>function binarySearchOnAnswer(check, low, high) {
>    while (low < high) {
>        const mid = Math.floor(low + (high - low) / 2);
>        if (check(mid)) {
>            high = mid; // Answer could be mid or smaller
>        } else {
>            low = mid + 1; // Need larger value
>        }
>    }
>    return low;
>}
>```

### Example: Koko Eating Bananas

```python
def minEatingSpeed(piles, h):
    def canFinish(speed):
        hours = sum((p + speed - 1) // speed for p in piles)
        return hours <= h

    left, right = 1, max(piles)

    while left < right:
        mid = left + (right - left) // 2
        if canFinish(mid):
            right = mid
        else:
            left = mid + 1

    return left
```

## Common Patterns

| Pattern | Use Case |
|---------|----------|
| Find exact value | Standard binary search |
| Find first occurrence | Lower bound |
| Find last occurrence | Upper bound |
| Find insertion point | bisect_left |
| Search on answer | Minimization/maximization |

## Complexity

- **Time**: O(log n)
- **Space**: O(1) iterative, O(log n) recursive

## Key Interview Problems

| Problem | Type | Difficulty | LeetCode Link |
| --------- | ------ | ------------ | --- |
| Binary Search | Basic | Easy | [Link](https://leetcode.com/problems/binary-search/) |
| Search Insert Position | Insertion Point | Easy | [Link](https://leetcode.com/problems/search-insert-position/) |
| First Bad Version | Lower Bound | Easy | [Link](https://leetcode.com/problems/first-bad-version/) |
| Search in Rotated Array | Modified | Medium | [Link](https://leetcode.com/problems/search-in-rotated-sorted-array/) |
| Find Peak Element | Mountain | Medium | [Link](https://leetcode.com/problems/find-peak-element/) |
| Koko Eating Bananas | On Answer | Medium | [Link](https://leetcode.com/problems/koko-eating-bananas/) |
| Median of Two Sorted Arrays | Advanced | Hard | [Link](https://leetcode.com/problems/median-of-two-sorted-arrays/) |
