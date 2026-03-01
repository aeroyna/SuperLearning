# Quick Select

Quick Select is an efficient algorithm to find the **Kth smallest** (or largest) element in an unordered list. It is related to Quick Sort but only recurses into one partition, achieving **O(N)** average time complexity.

## Concept

To find the Kth smallest element:
1.  **Partition**: Choose a `pivot` and partition the array such that elements smaller than pivot are on left, larger on right.
2.  **Check Position**: Let `p` be the final index of the pivot.
    - If `p == k`, we found the Kth element.
    - If `p > k`, the Kth element is in the left part. Recurse Left.
    - If `p < k`, the Kth element is in the right part. Recurse Right.

## Implementation (Kth Largest Element)

Finding Kth Largest is equivalent to finding `(N - K)`th Smallest (0-indexed).

>[!example]- C++
>```cpp
>int partition(vector<int>& nums, int left, int right) {
>    int pivot = nums[right];
>    int p = left;
>    for (int i = left; i < right; i++) {
>        if (nums[i] <= pivot) {
>            swap(nums[i], nums[p]);
>            p++;
>        }
>    }
>    swap(nums[p], nums[right]);
>    return p;
>}
>
>int quickSelect(vector<int>& nums, int k) {
>    int left = 0, right = nums.size() - 1;
>    int target = nums.size() - k; // Convert to Kth smallest index
>    
>    while (left <= right) {
>        int p = partition(nums, left, right);
>        if (p == target) return nums[p];
>        else if (p < target) left = p + 1;
>        else right = p - 1;
>    }
>    return -1;
>}
>```

>[!example]- Java
>```java
>public int findKthLargest(int[] nums, int k) {
>    int left = 0, right = nums.length - 1;
>    int target = nums.length - k;
>    
>    while (left <= right) {
>        int p = partition(nums, left, right);
>        if (p == target) return nums[p];
>        else if (p < target) left = p + 1;
>        else right = p - 1;
>    }
>    return -1;
>}
>
>private int partition(int[] nums, int left, int right) {
>    int pivot = nums[right];
>    int p = left;
>    for (int i = left; i < right; i++) {
>        if (nums[i] <= pivot) {
>            swap(nums, i, p);
>            p++;
>        }
>    }
>    swap(nums, p, right);
>    return p;
>}
>
>private void swap(int[] nums, int i, int j) {
>    int temp = nums[i];
>    nums[i] = nums[j];
>    nums[j] = temp;
>}
>```

>[!example]- Python
>```python
>def findKthLargest(nums, k):
>    target = len(nums) - k
>    
>    def partition(l, r):
>        pivot = nums[r]
>        p = l
>        for i in range(l, r):
>            if nums[i] <= pivot:
>                nums[i], nums[p] = nums[p], nums[i]
>                p += 1
>        nums[p], nums[r] = nums[r], nums[p]
>        return p
>    
>    l, r = 0, len(nums) - 1
>    while l <= r:
>        p = partition(l, r)
>        if p == target:
>            return nums[p]
>        elif p < target:
>            l = p + 1
>        else:
>            r = p - 1
>            
>    return -1
>```

>[!example]- JavaScript
>```javascript
>var findKthLargest = function(nums, k) {
>    const target = nums.length - k;
>    let left = 0;
>    let right = nums.length - 1;
>    
>    const partition = (l, r) => {
>        const pivot = nums[r];
>        let p = l;
>        for (let i = l; i < r; i++) {
>            if (nums[i] <= pivot) {
>                [nums[i], nums[p]] = [nums[p], nums[i]];
>                p++;
>            }
>        }
>        [nums[p], nums[r]] = [nums[r], nums[p]];
>        return p;
>    };
>    
>    while (left <= right) {
>        const p = partition(left, right);
>        if (p === target) return nums[p];
>        else if (p < target) left = p + 1;
>        else right = p - 1;
>    }
>    return -1;
>};
>```

## Pattern Recognition

**Signals**:
- "Find Kth smallest/largest element"
- "Find top K elements" (can be alternative to Heap)
- "Find median of unsorted array"
- O(N) constraint for sorting-like task

## Complexity Analysis

- **Time Complexity**: **O(N)** average case. **O(N^2)** worst case (sorted array with bad pivot). Random shuffling or median-of-medians prevents worst case.
- **Space Complexity**: **O(1)**. In-place partitioning.

## Common Problems

### 1. Kth Largest Element in an Array
Direct application.

### 2. K Closest Points to Origin
Compute distances, find Kth smallest distance, return all points with dist <= Kth dist.

### 3. Wiggle Sort II
Sort array such that `nums[0] < nums[1] > nums[2] < nums[3]...`.
- Can use Quick Select to find median, then map indices (Virtual Indexing) to place elements.

### Practice
- [Practice Problems](Practice_Problems/00_practice_problems.md)
