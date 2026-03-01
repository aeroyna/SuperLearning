# Practice Problems: K-Way Merge

Merging multiple sorted streams.

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Merge K Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/) | Hard | Min-heap of size K with list heads. |
| [Kth Smallest Element in a Sorted Matrix](https://leetcode.com/problems/kth-smallest-element-in-a-sorted-matrix/) | Medium | Min-heap of row heads or Binary Search on value range. |
| [Smallest Range Covering Elements from K Lists](https://leetcode.com/problems/smallest-range-covering-elements-from-k-lists/) | Hard | Heap of size K. Range is `max_in_heap - min_in_heap`. |
| [Find K Pairs with Smallest Sums](https://leetcode.com/problems/find-k-pairs-with-smallest-sums/) | Medium | Merge K implicit streams `(nums1[i], nums2[0])`. |
