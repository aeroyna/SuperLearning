## Interval Pattern: Merge Intervals

The "Merge Intervals" problem is a classic that serves as a foundation for many other, more complex interval-based questions.

### The Problem (LeetCode #56)
Given a collection of intervals, merge all overlapping intervals.

**Example**:
- `Input: [[1,3], [2,6], [8,10], [15,18]]`
- `Output: [[1,6], [8,10], [15,18]]`
- **Explanation**: The intervals `[1,3]` and `[2,6]` overlap and are merged into `[1,6]`.

### The "Sort by Start" Approach
The most intuitive and effective way to solve this problem is to first sort the intervals based on their **start times**. This ensures that as we iterate through the intervals, we are always looking at the next interval in chronological order.

**Algorithm**:
1.  **Sort**: Sort the array of intervals based on their start times.
2.  **Initialize**: Create a `merged_intervals` list and initialize it with the first interval from the sorted list.
3.  **Iterate and Merge**:
    - Iterate through the sorted intervals, starting from the second one.
    - For each `current_interval`, compare it with the `last_interval` in your `merged_intervals` list.
    - **Check for Overlap**: An overlap occurs if the `current_interval.start` is less than or equal to the `last_interval.end`.
    - **If they overlap**:
        - "Merge" them by updating the `last_interval.end` to be the maximum of `last_interval.end` and `current_interval.end`. This extends the merged interval to encompass the current one.
    - **If they do not overlap**:
        - There is a gap between the intervals. The `current_interval` starts a new, non-overlapping block. Add it to the `merged_intervals` list.
4.  Return the `merged_intervals` list.

### Implementation

>[!example]- C++
>```cpp
>class Solution {
>public:
>    vector<vector<int>> merge(vector<vector<int>>& intervals) {
>        // An empty or single-interval list is already merged.
>        if (intervals.empty() || intervals.size() < 2) {
>            return intervals;
>        }
>
>        // 1. Sort the intervals based on the start time.
>        sort(intervals.begin(), intervals.end(),
>             [](const vector<int>& a, const vector<int>& b) {
>                 return a[0] < b[0];
>             });
>
>        // 2. Initialize the merged list with the first interval.
>        vector<vector<int>> merged;
>        merged.push_back(intervals[0]);
>
>        // 3. Iterate from the second interval.
>        for (int i = 1; i < intervals.size(); i++) {
>            vector<int>& last_merged_interval = merged.back();
>            vector<int> current_interval = intervals[i];
>
>            // Check for overlap
>            // The end of the last merged interval is greater than or equal to
>            // the start of the current interval.
>            if (last_merged_interval[1] >= current_interval[0]) {
>                // Merge by updating the end of the last merged interval
>                last_merged_interval[1] = max(last_merged_interval[1], current_interval[1]);
>            } else {
>                // No overlap, so add the current interval as a new one.
>                merged.push_back(current_interval);
>            }
>        }
>
>        return merged;
>    }
>};
>
>// Example usage:
>// vector<vector<int>> intervals = {{1,3}, {8,10}, {2,6}, {15,18}};
>// Solution sol;
>// vector<vector<int>> result = sol.merge(intervals);
>// Result: [[1,6], [8,10], [15,18]]
>```

>[!example]- Java
>```java
>class Solution {
>    public int[][] merge(int[][] intervals) {
>        // An empty or single-interval list is already merged.
>        if (intervals == null || intervals.length < 2) {
>            return intervals;
>        }
>
>        // 1. Sort the intervals based on the start time.
>        Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]));
>
>        // 2. Initialize the merged list with the first interval.
>        List<int[]> merged = new ArrayList<>();
>        merged.add(intervals[0]);
>
>        // 3. Iterate from the second interval.
>        for (int i = 1; i < intervals.length; i++) {
>            int[] lastMergedInterval = merged.get(merged.size() - 1);
>            int[] currentInterval = intervals[i];
>
>            // Check for overlap
>            // The end of the last merged interval is greater than or equal to
>            // the start of the current interval.
>            if (lastMergedInterval[1] >= currentInterval[0]) {
>                // Merge by updating the end of the last merged interval
>                lastMergedInterval[1] = Math.max(lastMergedInterval[1], currentInterval[1]);
>            } else {
>                // No overlap, so add the current interval as a new one.
>                merged.add(currentInterval);
>            }
>        }
>
>        return merged.toArray(new int[merged.size()][]);
>    }
>}
>
>// Example usage:
>// int[][] intervals = {{1,3}, {8,10}, {2,6}, {15,18}};
>// Solution sol = new Solution();
>// int[][] result = sol.merge(intervals);
>// Result: [[1,6], [8,10], [15,18]]
>```

>[!example]- Python
>```python
>def merge_intervals(intervals):
>    # An empty or single-interval list is already merged.
>    if not intervals or len(intervals) < 2:
>        return intervals
>
>    # 1. Sort the intervals based on the start time.
>    intervals.sort(key=lambda x: x[0])
>
>    # 2. Initialize the merged list with the first interval.
>    merged = [intervals[0]]
>
>    # 3. Iterate from the second interval.
>    for i in range(1, len(intervals)):
>        last_merged_interval = merged[-1]
>        current_interval = intervals[i]
>
>        # Check for overlap
>        # The end of the last merged interval is greater than or equal to
>        # the start of the current interval.
>        if last_merged_interval[1] >= current_interval[0]:
>            # Merge by updating the end of the last merged interval
>            last_merged_interval[1] = max(last_merged_interval[1], current_interval[1])
>        else:
>            # No overlap, so add the current interval as a new one.
>            merged.append(current_interval)
>
>    return merged
>
># Example:
>intervals = [[1,3], [8,10], [2,6], [15,18]]
># Sorted: [[1,3], [2,6], [8,10], [15,18]]
># 1. merged = [[1,3]]
># 2. current = [2,6]. Overlap (3 >= 2). merged becomes [[1,6]].
># 3. current = [8,10]. No overlap (6 < 8). merged becomes [[1,6], [8,10]].
># 4. current = [15,18]. No overlap (10 < 15). merged becomes [[1,6], [8,10], [15,18]].
>print(merge_intervals(intervals))
>```

>[!example]- JavaScript
>```javascript
>/**
> * @param {number[][]} intervals
> * @return {number[][]}
> */
>function merge(intervals) {
>    // An empty or single-interval list is already merged.
>    if (!intervals || intervals.length < 2) {
>        return intervals;
>    }
>
>    // 1. Sort the intervals based on the start time.
>    intervals.sort((a, b) => a[0] - b[0]);
>
>    // 2. Initialize the merged list with the first interval.
>    const merged = [intervals[0]];
>
>    // 3. Iterate from the second interval.
>    for (let i = 1; i < intervals.length; i++) {
>        const lastMergedInterval = merged[merged.length - 1];
>        const currentInterval = intervals[i];
>
>        // Check for overlap
>        // The end of the last merged interval is greater than or equal to
>        // the start of the current interval.
>        if (lastMergedInterval[1] >= currentInterval[0]) {
>            // Merge by updating the end of the last merged interval
>            lastMergedInterval[1] = Math.max(lastMergedInterval[1], currentInterval[1]);
>        } else {
>            // No overlap, so add the current interval as a new one.
>            merged.push(currentInterval);
>        }
>    }
>
>    return merged;
>}
>
>// Example usage:
>// const intervals = [[1,3], [8,10], [2,6], [15,18]];
>// Sorted: [[1,3], [2,6], [8,10], [15,18]]
>// 1. merged = [[1,3]]
>// 2. current = [2,6]. Overlap (3 >= 2). merged becomes [[1,6]].
>// 3. current = [8,10]. No overlap (6 < 8). merged becomes [[1,6], [8,10]].
>// 4. current = [15,18]. No overlap (10 < 15). merged becomes [[1,6], [8,10], [15,18]].
>// console.log(merge(intervals));
>```

This O(n log n) approach (dominated by the sort) is efficient and robust.
