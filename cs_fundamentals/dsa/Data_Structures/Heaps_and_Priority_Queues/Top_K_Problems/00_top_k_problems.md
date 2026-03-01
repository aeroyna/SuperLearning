# Top K Problems

"Top K" problems ask for the K largest, smallest, or most frequent elements. Heaps provide an elegant O(n log k) solution that's often superior to the naive O(n log n) sorting approach, especially when k << n.

## Overview

Three main approaches:
1. **Sort then slice**: O(n log n) - simple but slow
2. **Heap of size K**: O(n log k) - optimal for streaming
3. **QuickSelect**: O(n) average - fastest for single query

## Topics

- [10.3.1 Top K Problems](01_top_k_problems.md)

## Core Technique: Bounded Heap

For K largest: maintain min-heap of size K. Elements smaller than heap's minimum can't be in top K.

>[!example]- C++
>```cpp
>vector<int> topKLargest(vector<int>& nums, int k) {
>    priority_queue<int, vector<int>, greater<int>> minHeap;
>    for (int num : nums) {
>        if (minHeap.size() < k) {
>            minHeap.push(num);
>        } else if (num > minHeap.top()) {
>            minHeap.pop();
>            minHeap.push(num);
>        }
>    }
>    vector<int> result;
>    while (!minHeap.empty()) {
>        result.push_back(minHeap.top());
>        minHeap.pop();
>    }
>    return result;
>}
>```

>[!example]- Java
>```java
>public List<Integer> topKLargest(int[] nums, int k) {
>    PriorityQueue<Integer> minHeap = new PriorityQueue<>();
>    for (int num : nums) {
>        if (minHeap.size() < k) {
>            minHeap.offer(num);
>        } else if (num > minHeap.peek()) {
>            minHeap.poll();
>            minHeap.offer(num);
>        }
>    }
>    return new ArrayList<>(minHeap);
>}
>```

>[!example]- Python
>```python
>import heapq
>
>def top_k_largest(nums, k):
>    # Min-heap of size k
>    heap = []
>    for num in nums:
>        if len(heap) < k:
>            heapq.heappush(heap, num)
>        elif num > heap[0]:  # Larger than smallest in heap
>            heapq.heapreplace(heap, num)  # Pop min, push new
>    return heap
>```

>[!example]- JavaScript
>```javascript
>// Requires MinPriorityQueue
>function topKLargest(nums, k) {
>    const minHeap = new MinPriorityQueue();
>    for (const num of nums) {
>        if (minHeap.size() < k) {
>            minHeap.enqueue(num);
>        } else if (num > minHeap.front().element) {
>            minHeap.dequeue();
>            minHeap.enqueue(num);
>        }
>    }
>    return minHeap.toArray().map(item => item.element);
>}
>```

**Why min-heap for largest**: The smallest element in the heap is the "gatekeeper"—anything smaller can't be in top K.

```
Finding top 3 from [7, 2, 9, 1, 5, 8, 3]:

Process 7: heap = [7]
Process 2: heap = [2, 7]
Process 9: heap = [2, 7, 9]
Process 1: 1 < 2 (min), skip
Process 5: 5 > 2, replace → heap = [5, 7, 9]
Process 8: 8 > 5, replace → heap = [7, 8, 9]
Process 3: 3 < 7 (min), skip

Result: [7, 8, 9]
```

## Pattern Variations

### K Smallest Elements

For K smallest: use max-heap of size K (negate values with heapq).

```python
def top_k_smallest(nums, k):
    # Max-heap (negated) of size k
    heap = []
    for num in nums:
        if len(heap) < k:
            heapq.heappush(heap, -num)
        elif num < -heap[0]:  # Smaller than largest in heap
            heapq.heapreplace(heap, -num)
    return [-x for x in heap]
```

### K Most Frequent

```python
from collections import Counter
import heapq

def top_k_frequent(nums, k):
    count = Counter(nums)

    # Min-heap of (frequency, element)
    heap = []
    for num, freq in count.items():
        if len(heap) < k:
            heapq.heappush(heap, (freq, num))
        elif freq > heap[0][0]:
            heapq.heapreplace(heap, (freq, num))

    return [num for freq, num in heap]
```

### K Closest Points to Origin

```python
def k_closest(points, k):
    # Max-heap of (-distance, point) to keep k smallest distances
    heap = []
    for x, y in points:
        dist = x*x + y*y  # Skip sqrt, relative comparison
        if len(heap) < k:
            heapq.heappush(heap, (-dist, x, y))
        elif -dist > heap[0][0]:  # Closer than furthest in heap
            heapq.heapreplace(heap, (-dist, x, y))

    return [[x, y] for _, x, y in heap]
```

## QuickSelect Alternative

For single query (not streaming), QuickSelect achieves O(n) average:

```python
def quickselect(nums, k):
    """Find kth smallest element."""
    def partition(left, right, pivot_idx):
        pivot = nums[pivot_idx]
        nums[pivot_idx], nums[right] = nums[right], nums[pivot_idx]
        store_idx = left
        for i in range(left, right):
            if nums[i] < pivot:
                nums[i], nums[store_idx] = nums[store_idx], nums[i]
                store_idx += 1
        nums[store_idx], nums[right] = nums[right], nums[store_idx]
        return store_idx

    left, right = 0, len(nums) - 1
    while left <= right:
        pivot_idx = random.randint(left, right)
        pivot_idx = partition(left, right, pivot_idx)

        if pivot_idx == k:
            return nums[k]
        elif pivot_idx < k:
            left = pivot_idx + 1
        else:
            right = pivot_idx - 1
```

## Approach Comparison

| Approach | Time | Space | Use Case |
|----------|------|-------|----------|
| Sort + slice | O(n log n) | O(1)* | Simple, k ≈ n |
| Heap size k | O(n log k) | O(k) | Streaming, k << n |
| QuickSelect | O(n) avg | O(1) | Single query, in-place OK |
| Bucket sort | O(n) | O(n) | Known value range |

*O(n) if sort not in-place

## Decision Framework

```
Is data streaming (elements arrive over time)?
├── Yes → Heap of size k
└── No → Can modify input array?
    ├── Yes → QuickSelect for one query
    └── No → Consider:
        ├── k close to n → Sort
        └── k << n → Heap
```

## Two Heaps Pattern

For problems like "find median":

```python
class MedianFinder:
    def __init__(self):
        self.small = []  # Max-heap (negated) for smaller half
        self.large = []  # Min-heap for larger half

    def add_num(self, num):
        heapq.heappush(self.small, -num)

        # Ensure all small ≤ all large
        if self.small and self.large and -self.small[0] > self.large[0]:
            heapq.heappush(self.large, -heapq.heappop(self.small))

        # Balance sizes (small can have at most 1 more)
        if len(self.small) > len(self.large) + 1:
            heapq.heappush(self.large, -heapq.heappop(self.small))
        if len(self.large) > len(self.small):
            heapq.heappush(self.small, -heapq.heappop(self.large))

    def find_median(self):
        if len(self.small) > len(self.large):
            return -self.small[0]
        return (-self.small[0] + self.large[0]) / 2
```

## Common Pitfalls

1. **Wrong heap type**: Min-heap for K largest, max-heap for K smallest
2. **Forgetting to bound heap size**: Heap grows unbounded if not careful
3. **Tuple comparison**: Ensure first element is the comparison key
4. **Ties in frequency problems**: Clarify secondary sort criteria

## Key Interview Problems

| Problem | Variant | Difficulty | LeetCode Link |
| --------- | --------- | ------------ | --- |
| Kth Largest Element | Basic | Medium | [Link](https://leetcode.com/problems/kth-largest-element-in-an-array/) |
| Top K Frequent Elements | Frequency | Medium | [Link](https://leetcode.com/problems/top-k-frequent-elements/) |
| K Closest Points to Origin | Distance metric | Medium | [Link](https://leetcode.com/problems/k-closest-points-to-origin/) |
| Find Median from Data Stream | Two heaps | Hard | [Link](https://leetcode.com/problems/find-median-from-data-stream/) |
| Merge K Sorted Lists | Multi-way merge | Hard | [Link](https://leetcode.com/problems/merge-k-sorted-lists/) |
| Ugly Number II | Generate sequence | Medium | [Link](https://leetcode.com/problems/ugly-number-ii/) |
