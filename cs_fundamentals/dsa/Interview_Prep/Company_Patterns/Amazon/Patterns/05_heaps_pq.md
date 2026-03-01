# Amazon Heaps & Priority Queues Patterns

**Frequency**: 🟠 **High**

Heaps and Priority Queues are frequently used at Amazon for problems involving "Top K" elements, merging sorted streams, and dynamically maintaining order in data streams.

## Key Concepts
- **Min-Heap / Max-Heap**: Maintaining smallest/largest elements.
- **K-way Merge**: Merging multiple sorted lists/arrays efficiently.
- **Two Heaps**: Often used to find median in a stream or similar problems.
- **Custom Comparators**: For complex object sorting in heaps.

## Phase 1: Must-Do (Foundation)

Master these 10 problems to build a solid foundation.

| Problem | Difficulty | Key Concept |
| :--- | :--- | :--- |
| [Kth Largest Element in an Array](https://leetcode.com/problems/kth-largest-element-in-an-array/) | Medium | Min-Heap or Quick Select. |
| [Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/) | Medium | Min-Heap to keep track of K largest frequencies. |
| [K Closest Points to Origin](https://leetcode.com/problems/k-closest-points-to-origin/) | Medium | Max-Heap to keep track of K smallest distances. |
| [Merge k Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/) | Hard | Min-Heap of list nodes. |
| [Task Scheduler](https://leetcode.com/problems/task-scheduler/) | Medium | Max-Heap or Greedy. |
| [Last Stone Weight](https://leetcode.com/problems/last-stone-weight/) | Easy | Max-Heap. |
| [Reorganize String](https://leetcode.com/problems/reorganize-string/) | Medium | Max-Heap (Greedy). |
| [Find K Pairs with Smallest Sums](https://leetcode.com/problems/find-k-pairs-with-smallest-sums/) | Medium | Min-Heap (K-way merge). |
| [Kth Largest Element in a Stream](https://leetcode.com/problems/kth-largest-element-in-a-stream/) | Easy | Min-Heap of size K. |
| [Sort Characters By Frequency](https://leetcode.com/problems/sort-characters-by-frequency/) | Medium | Max-Heap or Bucket Sort. |

## Phase 2: Practice & Variants (Depth)

Tackle these 10 harder variations and common follow-ups.

| Problem | Difficulty | Key Concept |
| :--- | :--- | :--- |
| [Find Median from Data Stream](https://leetcode.com/problems/find-median-from-data-stream/) | Hard | Two Heaps (one max, one min) of equal/near-equal size. |
| [Sliding Window Median](https://leetcode.com/problems/sliding-window-median/) | Hard | Two Heaps with Lazy Removal. |
| [Smallest Range Covering Elements from K Lists](https://leetcode.com/problems/smallest-range-covering-elements-from-k-lists/) | Hard | Min-Heap. |
| [IPO](https://leetcode.com/problems/ipo/) | Hard | Two Heaps (Max-Heap for profit, Min-Heap for capital). |
| [Course Schedule III](https://leetcode.com/problems/course-schedule-iii/) | Hard | Max-Heap (Greedy, remove longest duration). |
| [Minimum Cost to Hire K Workers](https://leetcode.com/problems/minimum-cost-to-hire-k-workers/) | Hard | Max-Heap (Quality/Wage ratio). |
| [Swim in Rising Water](https://leetcode.com/problems/swim-in-rising-water/) | Hard | Dijkstra (Min-Heap). |
| [Seat Reservation Manager](https://leetcode.com/problems/seat-reservation-manager/) | Medium | Min-Heap. |
| [Super Ugly Number](https://leetcode.com/problems/super-ugly-number/) | Medium | K-way merge with Min-Heap. |
| [Meeting Rooms III](https://leetcode.com/problems/meeting-rooms-iii/) | Hard | Min-Heap (Intervals, room availability). |
