# Binary Search Patterns

Binary search is more than just "find element in sorted array." Understanding its patterns unlocks many optimization problems.

## Basic Binary Search

```cpp
int binarySearch(std::vector<int>& nums, int target) {
    int left = 0, right = nums.size() - 1;

    while (left <= right) {
        int mid = left + (right - left) / 2;  // Avoid overflow

        if (nums[mid] == target) {
            return mid;
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }

    return -1;  // Not found
}
```

## Pattern 1: Find Boundary (Lower/Upper Bound)

### Lower Bound (First Position >= Target)

```cpp
int lowerBound(std::vector<int>& nums, int target) {
    int left = 0, right = nums.size();

    while (left < right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) {
            left = mid + 1;
        } else {
            right = mid;
        }
    }

    return left;  // First element >= target
}
// STL: std::lower_bound(nums.begin(), nums.end(), target)
```

### Upper Bound (First Position > Target)

```cpp
int upperBound(std::vector<int>& nums, int target) {
    int left = 0, right = nums.size();

    while (left < right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] <= target) {
            left = mid + 1;
        } else {
            right = mid;
        }
    }

    return left;  // First element > target
}
// STL: std::upper_bound(nums.begin(), nums.end(), target)
```

## Pattern 2: Binary Search on Answer

When you can't search the array directly, but can check "is answer X valid?"

```cpp
// Find minimum capacity to ship packages within D days
int shipWithinDays(std::vector<int>& weights, int days) {
    int left = *std::max_element(weights.begin(), weights.end());
    int right = std::accumulate(weights.begin(), weights.end(), 0);

    auto canShip = [&](int capacity) {
        int daysNeeded = 1, currentLoad = 0;
        for (int w : weights) {
            if (currentLoad + w > capacity) {
                ++daysNeeded;
                currentLoad = 0;
            }
            currentLoad += w;
        }
        return daysNeeded <= days;
    };

    while (left < right) {
        int mid = left + (right - left) / 2;
        if (canShip(mid)) {
            right = mid;
        } else {
            left = mid + 1;
        }
    }

    return left;
}
```

### Template for Binary Search on Answer

```cpp
int binarySearchOnAnswer(/* params */) {
    int left = MIN_POSSIBLE_ANSWER;
    int right = MAX_POSSIBLE_ANSWER;

    while (left < right) {
        int mid = left + (right - left) / 2;

        if (isValid(mid)) {
            right = mid;  // For finding minimum valid
            // left = mid;  // For finding maximum valid
        } else {
            left = mid + 1;  // For finding minimum valid
            // right = mid - 1;  // For finding maximum valid
        }
    }

    return left;
}
```

## Pattern 3: Rotated Sorted Array

```cpp
int searchRotated(std::vector<int>& nums, int target) {
    int left = 0, right = nums.size() - 1;

    while (left <= right) {
        int mid = left + (right - left) / 2;

        if (nums[mid] == target) return mid;

        // Left half is sorted
        if (nums[left] <= nums[mid]) {
            if (target >= nums[left] && target < nums[mid]) {
                right = mid - 1;
            } else {
                left = mid + 1;
            }
        }
        // Right half is sorted
        else {
            if (target > nums[mid] && target <= nums[right]) {
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }
    }

    return -1;
}
```

## Pattern 4: Find Peak Element

```cpp
int findPeakElement(std::vector<int>& nums) {
    int left = 0, right = nums.size() - 1;

    while (left < right) {
        int mid = left + (right - left) / 2;

        if (nums[mid] > nums[mid + 1]) {
            right = mid;  // Peak is on the left (including mid)
        } else {
            left = mid + 1;  // Peak is on the right
        }
    }

    return left;
}
```

## Pattern 5: Search in 2D Matrix

```cpp
bool searchMatrix(std::vector<std::vector<int>>& matrix, int target) {
    if (matrix.empty()) return false;

    int m = matrix.size(), n = matrix[0].size();
    int left = 0, right = m * n - 1;

    while (left <= right) {
        int mid = left + (right - left) / 2;
        int value = matrix[mid / n][mid % n];  // Convert to 2D

        if (value == target) return true;
        if (value < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }

    return false;
}
```

## Common Pitfalls

### 1. Integer Overflow
```cpp
// Bad: might overflow
int mid = (left + right) / 2;

// Good: no overflow
int mid = left + (right - left) / 2;
```

### 2. Infinite Loop
```cpp
// With left < right, use:
left = mid + 1;
right = mid;

// With left <= right, use:
left = mid + 1;
right = mid - 1;
```

### 3. Off-by-One Errors
```cpp
// When searching for boundary:
// - right = nums.size() (not size - 1)
// - return left (which equals right after loop)
```

## Key Takeaways

- Basic: find exact element in sorted array
- Boundary: find first/last position satisfying condition
- Answer search: when you can validate "is X possible?"
- Works on any monotonic sequence/condition
- Time: O(log n), Space: O(1)

## Common Interview Questions

> [!question]- When can you apply binary search?
> When there's a monotonic property: if condition(x) is true, then condition(x+1) is also true (or vice versa). The search space doesn't have to be an actual array.

> [!question]- How do you avoid infinite loops?
> Ensure the search space strictly decreases each iteration. Be careful with mid calculation and update logic.
