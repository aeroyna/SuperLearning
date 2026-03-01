# Practice Problems: Heaps

Priority queue management, top-k elements, and median finding.

## Top K Pattern

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Kth Largest Element in an Array](https://leetcode.com/problems/kth-largest-element-in-an-array/) | Medium | Min-heap of size K or QuickSelect. |
| [Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/) | Medium | Counter + Min-heap or Bucket Sort. |
| [K Closest Points to Origin](https://leetcode.com/problems/k-closest-points-to-origin/) | Medium | Max-heap of size K (keep smallest distance). |

## Heap Operations

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Merge K Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/) | Hard | Min-heap stores `(node.val, node)` from each list. |
| [Find Median from Data Stream](https://leetcode.com/problems/find-median-from-data-stream/) | Hard | Two heaps: Max-heap (lower half) & Min-heap (upper half). |
| [Task Scheduler](https://leetcode.com/problems/task-scheduler/) | Medium | Max-heap by frequency + Queue for cooldown. |
