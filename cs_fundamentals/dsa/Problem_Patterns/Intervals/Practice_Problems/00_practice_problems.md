# Practice Problems: Intervals

Sorting and merging ranges.

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Merge Intervals](https://leetcode.com/problems/merge-intervals/) | Medium | Sort by start. If `curr.start <= prev.end`, merge. |
| [Insert Interval](https://leetcode.com/problems/insert-interval/) | Medium | Add before, merge overlapping, add after. |
| [Non-overlapping Intervals](https://leetcode.com/problems/non-overlapping-intervals/) | Medium | Greedy strategy: Keep interval with earliest end time. |
| [Meeting Rooms II](https://leetcode.com/problems/meeting-rooms-ii/) | Medium | Sort starts and ends. Count active meetings. |
