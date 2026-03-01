# Line Sweep Algorithm

The Line Sweep (or Sweep Line) algorithm is a powerful visualization and computational technique used primarily in computational geometry and interval problems.

## Concept

Imagine a vertical line sweeping across a 2D plane from left to right (or top to bottom). As the line moves, it interacts with "events" (points, start of segments, end of segments). We process these events in sorted order to maintain the state of the active elements.

### Key Components

1.  **Events**: Discrete points where the state changes. For intervals `[start, end]`, the events are `start` (add interval) and `end` (remove interval).
2.  **Sorting**: Events are processed in increasing order of their coordinates.
3.  **Active Set (State)**: A data structure that keeps track of the objects currently intersecting the sweep line. This is often a BST, Heap, or simple counter.

## Implementation (Event Processing)

>[!example]- C++
>```cpp
>int maxOverlaps(vector<vector<int>>& intervals) {
>    vector<pair<int, int>> events;
>    for (const auto& interval : intervals) {
>        events.push_back({interval[0], 1});  // Start
>        events.push_back({interval[1], -1}); // End
>    }
>    
>    sort(events.begin(), events.end());
>    
>    int maxActive = 0, currentActive = 0;
>    for (const auto& event : events) {
>        currentActive += event.second;
>        maxActive = max(maxActive, currentActive);
>    }
>    return maxActive;
>}
>```

>[!example]- Java
>```java
>public int maxOverlaps(int[][] intervals) {
>    List<int[]> events = new ArrayList<>();
>    for (int[] interval : intervals) {
>        events.add(new int[]{interval[0], 1});  // Start
>        events.add(new int[]{interval[1], -1}); // End
>    }
>    
>    Collections.sort(events, (a, b) -> {
>        if (a[0] == b[0]) return a[1] - b[1]; // Process start before end if same time? Depends on problem.
>        return a[0] - b[0];
>    });
>    
>    int maxActive = 0, currentActive = 0;
>    for (int[] event : events) {
>        currentActive += event[1];
>        maxActive = Math.max(maxActive, currentActive);
>    }
>    return maxActive;
>}
>```

>[!example]- Python
>```python
>def max_overlaps(intervals):
>    events = []
>    for start, end in intervals:
>        events.append((start, 1))  # Start
>        events.append((end, -1))   # End
>    
>    events.sort()
>    
>    max_active = 0
>    current_active = 0
>    for _, change in events:
>        current_active += change
>        max_active = max(max_active, current_active)
>    
>    return max_active
>```

>[!example]- JavaScript
>```javascript
>function maxOverlaps(intervals) {
>    const events = [];
>    for (const [start, end] of intervals) {
>        events.push([start, 1]);  // Start
>        events.push([end, -1]);   // End
>    }
>    
>    events.sort((a, b) => {
>        if (a[0] === b[0]) return a[1] - b[1];
>        return a[0] - b[0];
>    });
>    
>    let maxActive = 0;
>    let currentActive = 0;
>    for (const [_, change] of events) {
>        currentActive += change;
>        maxActive = Math.max(maxActive, currentActive);
>    }
>    return maxActive;
>}
>```

## Pattern Recognition

**Signals**:
- "Maximum number of overlapping intervals"
- "Skyline problem"
- "Geometric intersections"
- "Number of airplanes in the sky"
- Coordinate range is large, but number of elements is manageable.

## Common Problems

### 1. Number of Airplanes in the Sky (Meeting Rooms II Variant)
Given a list of flight intervals `(start, end)`, find the maximum number of airplanes in the sky at the same time.
- **Approach**: Create events `(start, +1)` and `(end, -1)`. Sort events by time. If times are equal, process `+1` before `-1` (usually). Iterate and maintain a running sum. The answer is the max value of the sum.

### 2. The Skyline Problem
Given the positions and heights of buildings, return the "skyline" formed by these buildings collectively.
- **Approach**: Events are `(left_edge, -height)` and `(right_edge, height)`. Use a `Max-Heap` or `TreeMap` to store heights of "active" buildings. When moving across the critical points, if the max height changes, record a key point.

### 3. Rectangles Area Union
Find the total area covered by a set of rectangles.
- **Approach**: Sweep a vertical line. The "active length" of the vertical cut changes as we hit left/right edges of rectangles. Area = `active_length * (x_current - x_prev)`.

## Complexity Analysis

- **Time Complexity**: **O(N log N)**. Sorting the events takes `N log N`. Processing each event usually takes `O(1)` or `O(log N)` depending on the data structure used for the active set.
- **Space Complexity**: **O(N)** to store the events.

## Difference vs. Difference Array
- **Difference Array**: Best for static array updates where the range is fixed and small enough (e.g., array size 10^5).
- **Line Sweep**: Best when coordinates are sparse or very large (e.g., timestamps, 1 to 10^9), as it only processes the critical points.


### Practice
- [Practice Problems](Practice_Problems/00_practice_problems.md)