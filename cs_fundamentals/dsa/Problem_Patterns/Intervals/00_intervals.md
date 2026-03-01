## Introduction to Interval Problems

Interval problems are a common category of questions in coding interviews that involve reasoning about a set of intervals. An interval is typically represented as a pair of numbers `[start, end]`. These problems test your ability to handle overlapping ranges and often require a greedy approach, usually preceded by a custom sort.

### The Core Pattern
Most interval problems can be solved by following a consistent pattern:

1.  **Sort the Intervals**: This is almost always the first and most critical step. The way you sort the intervals depends on the specific problem. Common sorting strategies include:
    - Sorting by the **start time**.
    - Sorting by the **end time**.
    - Sometimes a secondary sorting key is needed for tie-breaking.

2.  **Iterate and Process**: After sorting, you iterate through the intervals and apply a specific logic (often greedy) to make decisions. This usually involves comparing the current interval with the previous one or with a result you are building up.

### Common Interval Problems

Two of the most classic interval problems that form the basis for many others are:

1.  **Merge Intervals**: Given a collection of intervals, merge all overlapping intervals into a set of mutually exclusive intervals.
    - **Key Idea**: Sort by start time. Iterate and merge the current interval with the previous one if they overlap.

2.  **Meeting Rooms**: This comes in two main variations:
    - **Meeting Rooms I**: Given a set of meeting time intervals, determine if a person could attend all meetings (i.e., if any two intervals overlap).
    - **Meeting Rooms II**: Given a set of meeting time intervals, find the minimum number of conference rooms required to hold all the meetings.
    - **Key Idea**: These problems often involve tracking start and end times to see how many "active" events are happening at any given moment.

Mastering the "sort and process" pattern is the key to solving most interval-based questions.


### Practice
- [Practice Problems](Practice_Problems/00_practice_problems.md)