## Greedy Algorithms: Interval Scheduling

The Interval Scheduling problem is a classic example that beautifully illustrates the core principles of greedy algorithms. It's a problem that appears in many forms in coding interviews.

### The Problem
You are given a collection of intervals, where each interval has a `start` time and an `end` time. The goal is to find the **maximum number of non-overlapping intervals** you can select from this collection.

**Example**: Given intervals `[[1, 3], [2, 4], [3, 6], [1, 2]]`
- If you select `[1, 3]`, you cannot select `[2, 4]` or `[1, 2]`. You could then select `[3, 6]`. Total: 2 intervals.
- If you select `[1, 2]`, you can then select `[2, 4]` (if intervals can touch at the boundary) or `[3, 6]`.
- The optimal solution is `[1, 2]` and `[3, 6]`, for a total of 2. (Another is `[1,2]` and `[2,4]`).

### Exploring Greedy Strategies
When faced with this problem, several intuitive greedy strategies come to mind. Let's see why most of them fail.
- **Greedy by Start Time**: "Always pick the interval that starts earliest."
  - **Counterexample**: `[[1, 10], [2, 3], [4, 5]]`. This strategy would pick `[1, 10]` first, which would block out the other two. The optimal solution is to pick `[2, 3]` and `[4, 5]`. This fails.
- **Greedy by Shortest Interval**: "Always pick the shortest interval first."
  - **Counterexample**: `[[4, 5], [1, 6], [7, 8]]`. This strategy would pick the short `[4, 5]` first. But this interval overlaps with `[1, 6]`, so picking it might not be optimal. In this case, `[1, 6]` blocks both others. `[[4,5], [7,8]]` is the optimal solution.
  Let's try a better counterexample `[[1,2], [3,8], [2,4]]`. Shortest is `[1,2]`. Then we can't take `[2,4]`, but we can take `[3,8]`. Total 2. What if we took `[2,4]`? Then we can't take `[1,2]` or `[3,8]`. Total 1. So shortest interval seems to work here. Let's try `[[3,4], [1,5], [2,6]]`. Shortest is `[3,4]`. Then we can't take `[1,5]` or `[2,6]`. Total 1. Optimal is just `[3,4]`. Another case `[[1,10], [2,3], [4,5]]`. Shortest are `[2,3]` and `[4,5]`. If we pick `[2,3]`, we can pick `[4,5]`. Total 2. If we pick `[1,10]`, total 1. The shortest interval strategy seems plausible but is more complex to prove and can be non-trivial.

### The Winning Strategy: Earliest Finish Time
The correct and most elegant greedy strategy is to **always pick the interval that finishes earliest.**

- **Algorithm**:
  1.  Sort the intervals based on their **finish times** in ascending order.
  2.  Initialize your solution by selecting the very first interval in the sorted list.
  3.  Iterate through the rest of the sorted intervals. For each interval, if its `start` time is greater than or equal to the `finish` time of the last interval you selected, then it doesn't overlap. Select this interval and update the "last finish time" to be the finish time of this new interval.

- **Proof of Correctness (Intuition)**: By choosing the interval that finishes earliest, you free up the resource (e.g., the meeting room, the time slot) as quickly as possible. This maximizes the remaining time for other potential intervals to be scheduled. Any other choice would involve picking an interval that finishes later, which could only reduce the opportunity for scheduling more events.

### Implementation

>[!example]- C++
>```cpp
>#include <vector>
>#include <algorithm>
>using namespace std;
>
>int maxNonOverlappingIntervals(vector<vector<int>>& intervals) {
>    if (intervals.empty()) {
>        return 0;
>    }
>
>    // Sort intervals by finish time
>    sort(intervals.begin(), intervals.end(),
>         [](const vector<int>& a, const vector<int>& b) {
>             return a[1] < b[1];
>         });
>
>    int count = 1;
>    int lastFinishTime = intervals[0][1];
>
>    for (int i = 1; i < intervals.size(); i++) {
>        int currentStartTime = intervals[i][0];
>
>        if (currentStartTime >= lastFinishTime) {
>            count++;
>            lastFinishTime = intervals[i][1];
>        }
>    }
>
>    return count;
>}
>
>// Example:
>// vector<vector<int>> intervals = {{1, 3}, {2, 4}, {3, 6}, {1, 2}};
>// cout << maxNonOverlappingIntervals(intervals) << endl;
>```

>[!example]- Java
>```java
>import java.util.*;
>
>public class IntervalScheduling {
>    public static int maxNonOverlappingIntervals(int[][] intervals) {
>        if (intervals == null || intervals.length == 0) {
>            return 0;
>        }
>
>        // Sort intervals by finish time
>        Arrays.sort(intervals, (a, b) -> Integer.compare(a[1], b[1]));
>
>        int count = 1;
>        int lastFinishTime = intervals[0][1];
>
>        for (int i = 1; i < intervals.length; i++) {
>            int currentStartTime = intervals[i][0];
>
>            if (currentStartTime >= lastFinishTime) {
>                count++;
>                lastFinishTime = intervals[i][1];
>            }
>        }
>
>        return count;
>    }
>
>    // Example:
>    // public static void main(String[] args) {
>    //     int[][] intervals = {{1, 3}, {2, 4}, {3, 6}, {1, 2}};
>    //     System.out.println(maxNonOverlappingIntervals(intervals));
>    // }
>}
>```

>[!example]- Python
>```python
>def max_non_overlapping_intervals(intervals):
>    if not intervals:
>        return 0
>
>    # Sort the intervals by their finish times
>    intervals.sort(key=lambda x: x[1])
>
>    count = 1
>    last_finish_time = intervals[0][1]
>
>    # Iterate and make the greedy choice
>    for i in range(1, len(intervals)):
>        current_start_time = intervals[i][0]
>
>        # If the current interval does not overlap with the last selected one
>        if current_start_time >= last_finish_time:
>            count += 1
>            last_finish_time = intervals[i][1]
>
>    return count
>
># Example:
>intervals = [[1, 3], [2, 4], [3, 6], [1, 2]]
>print(max_non_overlapping_intervals(intervals))
>```

>[!example]- JavaScript
>```javascript
>function maxNonOverlappingIntervals(intervals) {
>    if (!intervals || intervals.length === 0) {
>        return 0;
>    }
>
>    // Sort intervals by finish time
>    intervals.sort((a, b) => a[1] - b[1]);
>
>    let count = 1;
>    let lastFinishTime = intervals[0][1];
>
>    for (let i = 1; i < intervals.length; i++) {
>        const currentStartTime = intervals[i][0];
>
>        if (currentStartTime >= lastFinishTime) {
>            count++;
>            lastFinishTime = intervals[i][1];
>        }
>    }
>
>    return count;
>}
>
>// Example:
>const intervals = [[1, 3], [2, 4], [3, 6], [1, 2]];
>console.log(maxNonOverlappingIntervals(intervals));
>```
