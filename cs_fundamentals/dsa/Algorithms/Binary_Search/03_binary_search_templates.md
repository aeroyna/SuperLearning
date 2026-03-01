## Binary Search Templates

Implementing binary search can be tricky. Off-by-one errors are common, and small changes in the implementation (`<` vs `<=`, `right = mid` vs `right = mid - 1`) can lead to incorrect results or infinite loops. Having a few robust, well-understood templates can save a lot of time and prevent bugs.

### Template 1: Basic Search
This is the most common template, used to find if a specific `target` exists in a sorted array.

- **Search Space**: Inclusive range `[left, right]`.
- **Condition**: `while left <= right`. The `=` is important to handle the case where the search space has only one element.
- **Shrinking Logic**: The `mid` element is checked and then excluded from the next search space, i.e., `left = mid + 1` or `right = mid - 1`.

>[!example]- C++
>```cpp
>#include <vector>
>using namespace std;
>
>int binarySearch(vector<int>& nums, int target) {
>    int left = 0;
>    int right = nums.size() - 1;
>
>    while (left <= right) {
>        int mid = left + (right - left) / 2;
>
>        if (nums[mid] == target) {
>            return mid; // Target found
>        } else if (nums[mid] < target) {
>            left = mid + 1;
>        } else {
>            right = mid - 1;
>        }
>    }
>
>    return -1; // Target not found
>}
>```

>[!example]- Java
>```java
>public int binarySearch(int[] nums, int target) {
>    int left = 0;
>    int right = nums.length - 1;
>
>    while (left <= right) {
>        int mid = left + (right - left) / 2;
>
>        if (nums[mid] == target) {
>            return mid; // Target found
>        } else if (nums[mid] < target) {
>            left = mid + 1;
>        } else {
>            right = mid - 1;
>        }
>    }
>
>    return -1; // Target not found
>}
>```

>[!example]- Python
>```python
>def binary_search(nums, target):
>    left, right = 0, len(nums) - 1
>
>    while left <= right:
>        mid = left + (right - left) // 2
>
>        if nums[mid] == target:
>            return mid # Target found
>        elif nums[mid] < target:
>            left = mid + 1
>        else:
>            right = mid - 1
>
>    return -1 # Target not found
>```

>[!example]- JavaScript
>```javascript
>function binarySearch(nums, target) {
>    let left = 0;
>    let right = nums.length - 1;
>
>    while (left <= right) {
>        let mid = left + Math.floor((right - left) / 2);
>
>        if (nums[mid] === target) {
>            return mid; // Target found
>        } else if (nums[mid] < target) {
>            left = mid + 1;
>        } else {
>            right = mid - 1;
>        }
>    }
>
>    return -1; // Target not found
>}
>```

**Use When**: You need to find an exact value in a simple sorted array.

---

### Template 2: Finding a Boundary (Lower/Upper Bound)
This template is more versatile and is used to find the first element that meets a certain condition. It's excellent for finding the "insertion point," "lower bound," or the start of a duplicate range.

- **Search Space**: The key idea is to search over the space `[0, n]`. `left` points to the first possible element, and `right` points one past the last possible element.
- **Condition**: `while left < right`. We stop when `left == right`, at which point they both point to the desired boundary.
- **Shrinking Logic**: We are trying to find the *first* `mid` that satisfies our condition.
  - If `nums[mid]` meets the condition, we know `mid` is a *potential* answer, but there might be an earlier one. So we set `right = mid`, including `mid` in the future search space.
  - If `nums[mid]` does not meet the condition, we know `mid` and everything before it is not the answer, so we set `left = mid + 1`.

#### Example: Finding the Lower Bound (First occurrence of `target` or insertion point)
Find the first index `i` such that `nums[i] >= target`.

>[!example]- C++
>```cpp
>#include <vector>
>using namespace std;
>
>int lowerBound(vector<int>& nums, int target) {
>    int left = 0;
>    int right = nums.size(); // Search space is [0, n]
>
>    while (left < right) {
>        int mid = left + (right - left) / 2;
>
>        if (nums[mid] >= target) {
>            // mid is a potential answer, try to find an earlier one
>            right = mid;
>        } else {
>            // mid is too small, discard it and everything to its left
>            left = mid + 1;
>        }
>    }
>
>    // left is the insertion point or the first occurrence
>    return left;
>}
>```

>[!example]- Java
>```java
>public int lowerBound(int[] nums, int target) {
>    int left = 0;
>    int right = nums.length; // Search space is [0, n]
>
>    while (left < right) {
>        int mid = left + (right - left) / 2;
>
>        if (nums[mid] >= target) {
>            // mid is a potential answer, try to find an earlier one
>            right = mid;
>        } else {
>            // mid is too small, discard it and everything to its left
>            left = mid + 1;
>        }
>    }
>
>    // left is the insertion point or the first occurrence
>    return left;
>}
>```

>[!example]- Python
>```python
>def lower_bound(nums, target):
>    left, right = 0, len(nums) # Search space is [0, n]
>
>    while left < right:
>        mid = left + (right - left) // 2
>
>        if nums[mid] >= target:
>            # mid is a potential answer, try to find an earlier one
>            right = mid
>        else:
>            # mid is too small, discard it and everything to its left
>            left = mid + 1
>
>    # left is the insertion point or the first occurrence
>    return left
>```

>[!example]- JavaScript
>```javascript
>function lowerBound(nums, target) {
>    let left = 0;
>    let right = nums.length; // Search space is [0, n]
>
>    while (left < right) {
>        let mid = left + Math.floor((right - left) / 2);
>
>        if (nums[mid] >= target) {
>            // mid is a potential answer, try to find an earlier one
>            right = mid;
>        } else {
>            // mid is too small, discard it and everything to its left
>            left = mid + 1;
>        }
>    }
>
>    // left is the insertion point or the first occurrence
>    return left;
>}
>```

**Use When**: You need to find the first element that is `>=` some value, or the last element that is `<=` some value (with a slight modification). This template is very robust for "Binary Search on the Answer" problems as well. The final answer is almost always `left`.

---

### Choosing a Template
- **Template 1** is simple and great for basic existence checks. Its weakness is that it's less adaptable to boundary-finding problems.
- **Template 2** is more powerful and general. By carefully defining the condition in the `if` statement, you can solve a wide range of problems, including lower/upper bound, insertion position, and binary search on the answer. When in doubt, this template is often a good choice.
