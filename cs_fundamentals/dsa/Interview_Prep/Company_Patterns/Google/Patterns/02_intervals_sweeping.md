# Google Interval & Sweeping Patterns

**Frequency**: 🟠 **High**

Interval problems are a staple at Google. They test your ability to handle scheduling, resource allocation, and timeline events efficiently.

## Key Concepts
- **Sorting**: Almost always the first step (by start time or end time).
- **Line Sweep**: Processing events (start/end points) in chronological order to track the state of "active" intervals.
- **Min-Heap**: Efficiently tracking the end times of active intervals (e.g., Meeting Rooms II).
- **TreeMap**: Useful for dynamic interval problems where you need to insert/query efficiently (My Calendar).

## Phase 1: Must-Do (Foundation)

Master these 10 problems to build a solid foundation in Interval questions.

| Problem | Difficulty | Key Concept |
| :--- | :--- | :--- |
| [Merge Intervals](https://leetcode.com/problems/merge-intervals/) | Medium | Sort by start time. Merge if overlaps. |
| [Insert Interval](https://leetcode.com/problems/insert-interval/) | Medium | Binary search or linear scan. Handle overlap. |
| [Non-overlapping Intervals](https://leetcode.com/problems/non-overlapping-intervals/) | Medium | Greedy (Sort by end time). |
| [Meeting Rooms](https://leetcode.com/problems/meeting-rooms/) | Easy | Sort and check for any overlap. |
| [Meeting Rooms II](https://leetcode.com/problems/meeting-rooms-ii/) | Medium | Min-Heap or Line Sweep. |
| [Interval List Intersections](https://leetcode.com/problems/interval-list-intersections/) | Medium | Two pointers. |
| [Minimum Number of Arrows to Burst Balloons](https://leetcode.com/problems/minimum-number-of-arrows-to-burst-balloons/) | Medium | Greedy (Sort by end time). |
| [Teemo Attacking](https://leetcode.com/problems/teemo-attacking/) | Easy | Merge overlapping durations. |
| [Partition Labels](https://leetcode.com/problems/partition-labels/) | Medium | Greedy (Track last occurrence of char). |
| [Summary Ranges](https://leetcode.com/problems/summary-ranges/) | Easy | Linear scan for contiguous sequences. |

## Phase 2: Practice & Variants (Depth)

Tackle these 10 harder variations to handle edge cases and advanced constraints.

| Problem | Difficulty | Key Concept |
| :--- | :--- | :--- |
| [Employee Free Time](https://leetcode.com/problems/employee-free-time/) | Hard | Line Sweep / Merge logic. |
| [Data Stream as Disjoint Intervals](https://leetcode.com/problems/data-stream-as-disjoint-intervals/) | Hard | TreeMap (Java) / Ordered Set to merge on insert. |
| [My Calendar I](https://leetcode.com/problems/my-calendar-i/) | Medium | TreeMap (Balanced BST) for O(log N). |
| [My Calendar II](https://leetcode.com/problems/my-calendar-ii/) | Medium | Line Sweep or tracking double overlaps. |
| [My Calendar III](https://leetcode.com/problems/my-calendar-iii/) | Hard | Segment Tree or Line Sweep (Max active). |
| [The Skyline Problem](https://leetcode.com/problems/the-skyline-problem/) | Hard | Line Sweep + Max-Heap/TreeMap. |
| [Range Module](https://leetcode.com/problems/range-module/) | Hard | TreeMap (Dynamic removal/addition of intervals). |
| [Remove Covered Intervals](https://leetcode.com/problems/remove-covered-intervals/) | Medium | Sort by start, then end (descending). |
| [Maximum Number of Events That Can Be Attended](https://leetcode.com/problems/maximum-number-of-events-that-can-be-attended/) | Hard | Priority Queue + Greedy (Sort events by start). |
| [Minimum Interval to Include Each Query](https://leetcode.com/problems/minimum-interval-to-include-each-query/) | Hard | Sorting Queries + Min-Heap (Sweep Line). |