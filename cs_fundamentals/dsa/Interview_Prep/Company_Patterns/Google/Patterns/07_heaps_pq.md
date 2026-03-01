# Google Heaps & Priority Queues Patterns

**Frequency**: 🟡 **Medium**

Heap-based problems are common for optimizing selections, merging sorted collections, and managing priorities.

## Key Concepts
- **Min-Heap / Max-Heap**: Maintaining smallest/largest elements.
- **K-way Merge**: Merging multiple sorted lists/arrays.
- **Two Heaps**: Often used to find median in a stream or similar problems.

## Phase 1: Must-Do (Foundation)

Master these 10 problems to build a solid foundation.

| Problem | Difficulty | Key Concept |
| :--- | :--- | :--- |
| [Kth Largest Element in an Array](https://leetcode.com/problems/kth-largest-element-in-an-array/) | Medium | Min-Heap or Quick Select. |
| [Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/) | Medium | Min-Heap to keep track of K largest frequencies. |
| [K Closest Points to Origin](https://leetcode.com/problems/k-closest-points-to-origin/) | Medium | Max-Heap to keep track of K smallest distances. |
| [Merge K Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/) | Hard | Min-Heap of list nodes. |
| [Task Scheduler](https://leetcode.com/problems/task-scheduler/) | Medium | Max-Heap or Greedy. |
| [Last Stone Weight](https://leetcode.com/problems/last-stone-weight/) | Easy | Max-Heap. |
| [Reorganize String](https://leetcode.com/problems/reorganize-string/) | Medium | Max-Heap (Greedy). |
| [Find K Pairs with Smallest Sums](https://leetcode.com/problems/find-k-pairs-with-smallest-sums/) | Medium | Min-Heap (K-way merge). |
| [Kth Largest Element in a Stream](https://leetcode.com/problems/kth-largest-element-in-a-stream/) | Easy | Min-Heap of size K. |
| [Sort Characters By Frequency](https://leetcode.com/problems/sort-characters-by-frequency/) | Medium | Max-Heap or Bucket Sort. |

## Phase 2: Practice & Variants (Depth)

Tackle these 10 harder variations.

| Problem | Difficulty | Key Concept |
| :--- | :--- | :--- |
| [Find Median from Data Stream](https://leetcode.com/problems/find-median-from-data-stream/) | Hard | Two Heaps (one max, one min) of equal/near-equal size. |
| [Sliding Window Median](https://leetcode.com/problems/sliding-window-median/) | Hard | Two Heaps with Lazy Removal. |
| [The Skyline Problem](https://leetcode.com/problems/the-skyline-problem/) | Hard | Max-Heap to track active building heights. |
| [Trapping Rain Water II](https://leetcode.com/problems/trapping-rain-water-ii/) | Hard | Min-Heap (BFS from boundary). |
| [Employee Free Time](https://leetcode.com/problems/employee-free-time/) | Hard | Min-Heap of intervals or events. |
| [IPO](https://leetcode.com/problems/ipo/) | Hard | Two Heaps (Max-Heap for profit, Min-Heap for capital). |
| [Course Schedule III](https://leetcode.com/problems/course-schedule-iii/) | Hard | Max-Heap (Greedy, remove longest duration). |
| [Minimum Cost to Hire K Workers](https://leetcode.com/problems/minimum-cost-to-hire-k-workers/) | Hard | Max-Heap (Quality/Wage ratio). |
| [Swim in Rising Water](https://leetcode.com/problems/swim-in-rising-water/) | Hard | Dijkstra (Min-Heap). |
| [Design Twitter](https://leetcode.com/problems/design-twitter/) | Medium | Hash Map + Heap (Merge K Sorted). |