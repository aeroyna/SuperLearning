## Binary Search on Arrays

The most direct application of binary search is to find an element in a **sorted array**. Its O(log n) time complexity is a massive improvement over the O(n) required for a linear scan.

### The Core Pattern
The algorithm maintains two pointers, `left` and `right`, which define the current search space within the array.
1.  Initialize `left = 0` and `right = arr.length - 1`.
2.  While `left <= right`:
    a. Find the middle index: `mid = left + (right - left) // 2`. (This version avoids potential overflow).
    b. Compare the element at `arr[mid]` with the `target`.
    c. - If `arr[mid] == target`, the element is found.
       - If `arr[mid] < target`, the target must be in the right half, so update `left = mid + 1`.
       - If `arr[mid] > target`, the target must be in the left half, so update `right = mid - 1`.
3.  If the loop finishes without finding the element, it is not in the array.

### Variations for Interviews
In interviews, you'll often be asked to solve a variation of the basic search. The key is to slightly modify the binary search logic, especially how the pointers are updated and what the function returns.

#### 1. Finding the Insertion Point
If the target is not in the array, a standard binary search can be used to find the correct index where the target *should be inserted* to maintain sorted order. When the loop terminates (`left > right`), the `left` pointer will be at the correct insertion position.

#### 2. Finding First or Last Occurrence (Lower/Upper Bound)
When an array contains duplicate elements, you might need to find the first (leftmost) or last (rightmost) occurrence of a target. This requires a modified binary search.

- **To find the first occurrence (Lower Bound)**: Even when you find the target, you continue searching in the left half to see if an earlier occurrence exists.
  - `if arr[mid] >= target`: `right = mid - 1`
  - `else`: `left = mid + 1`

- **To find the first element strictly greater than target (Upper Bound)**:
  - `if arr[mid] > target`: `right = mid - 1`
  - `else`: `left = mid + 1`

These variations are crucial for problems that involve counting occurrences or finding elements within a certain range.

### Example: Search a 2D Matrix (LeetCode #74)
**Problem**: Given an `m x n` matrix where each row is sorted from left to right and the first integer of each row is greater than the last integer of the previous row, check if a `target` is present.

**Insight**: The properties of the matrix mean that if you "flatten" it into a 1D array, it would be completely sorted. This allows you to apply binary search over the entire matrix.

**Solution**:
1. Treat the `m * n` matrix as a virtual 1D array of size `m * n`.
2. The `left` pointer is `0` and the `right` pointer is `m * n - 1`.
3. In each step of the binary search, calculate the `mid` index for the virtual array.
4. Convert the 1D `mid` index back to a 2D `(row, col)` index to retrieve the value.
   - `row = mid // n` (integer division by column count)
   - `col = mid % n` (modulo by column count)
5. Proceed with the standard binary search comparison.

> [!example]- C++
> ```cpp
> class Solution {
> public:
>     bool searchMatrix(vector<vector<int>>& matrix, int target) {
>         if (matrix.empty() || matrix[0].empty()) {
>             return false;
>         }
>
>         int m = matrix.size(), n = matrix[0].size();
>         int left = 0, right = m * n - 1;
>
>         while (left <= right) {
>             int mid_idx = left + (right - left) / 2;
>             int mid_val = matrix[mid_idx / n][mid_idx % n];
>
>             if (mid_val == target) {
>                 return true;
>             } else if (mid_val < target) {
>                 left = mid_idx + 1;
>             } else {
>                 right = mid_idx - 1;
>             }
>         }
>
>         return false;
>     }
> };
> ```

> [!example]- Java
> ```java
> class Solution {
>     public boolean searchMatrix(int[][] matrix, int target) {
>         if (matrix == null || matrix.length == 0 || matrix[0].length == 0) {
>             return false;
>         }
>
>         int m = matrix.length, n = matrix[0].length;
>         int left = 0, right = m * n - 1;
>
>         while (left <= right) {
>             int midIdx = left + (right - left) / 2;
>             int midVal = matrix[midIdx / n][midIdx % n];
>
>             if (midVal == target) {
>                 return true;
>             } else if (midVal < target) {
>                 left = midIdx + 1;
>             } else {
>                 right = midIdx - 1;
>             }
>         }
>
>         return false;
>     }
> }
> ```

> [!example]- Python
> ```python
> def search_matrix(matrix, target):
>     if not matrix or not matrix[0]:
>         return False
>
>     m, n = len(matrix), len(matrix[0])
>     left, right = 0, m * n - 1
>
>     while left <= right:
>         mid_idx = left + (right - left) // 2
>         mid_val = matrix[mid_idx // n][mid_idx % n]
>
>         if mid_val == target:
>             return True
>         elif mid_val < target:
>             left = mid_idx + 1
>         else:
>             right = mid_idx - 1
>
>     return False
> ```

> [!example]- JavaScript
> ```javascript
> function searchMatrix(matrix, target) {
>     if (!matrix || matrix.length === 0 || matrix[0].length === 0) {
>         return false;
>     }
>
>     const m = matrix.length, n = matrix[0].length;
>     let left = 0, right = m * n - 1;
>
>     while (left <= right) {
>         const midIdx = left + Math.floor((right - left) / 2);
>         const midVal = matrix[Math.floor(midIdx / n)][midIdx % n];
>
>         if (midVal === target) {
>             return true;
>         } else if (midVal < target) {
>             left = midIdx + 1;
>         } else {
>             right = midIdx - 1;
>         }
>     }
>
>     return false;
> }
> ```
This is a classic example of adapting the binary search algorithm to a structure that is not explicitly a 1D array.
