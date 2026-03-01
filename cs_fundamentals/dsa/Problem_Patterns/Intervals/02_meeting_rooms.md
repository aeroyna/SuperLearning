## Interval Pattern: Meeting Rooms

The "Meeting Rooms" problems are another classic set of interval questions that build on the "sort and process" pattern. They are about managing resource allocation (in this case, rooms) based on time intervals.

---

### Meeting Rooms I (LeetCode #252)
**Problem**: Given an array of meeting time intervals, determine if a person could attend all meetings.

**Insight**: This is equivalent to asking, "Do any two intervals overlap?" If there is any overlap, one person cannot attend all meetings.

**Algorithm**:
1.  **Sort**: Sort the intervals by their **start times**.
2.  **Iterate and Check**: Loop through the sorted intervals from the second one.
3.  For each interval, check if its `start` time is less than the `end` time of the *previous* interval. If it is, an overlap exists, and you can immediately return `False`.
4.  If the loop completes without finding any overlaps, return `True`.

#### Implementation

>[!example]- C++
>```cpp
>class Solution {
>public:
>    bool canAttendMeetings(vector<vector<int>>& intervals) {
>        // Sort by start time
>        sort(intervals.begin(), intervals.end(),
>             [](const vector<int>& a, const vector<int>& b) {
>                 return a[0] < b[0];
>             });
>
>        for (int i = 1; i < intervals.size(); i++) {
>            // Check for overlap with the previous meeting
>            if (intervals[i][0] < intervals[i-1][1]) {
>                return false;
>            }
>        }
>
>        return true;
>    }
>};
>```

>[!example]- Java
>```java
>class Solution {
>    public boolean canAttendMeetings(int[][] intervals) {
>        // Sort by start time
>        Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]));
>
>        for (int i = 1; i < intervals.length; i++) {
>            // Check for overlap with the previous meeting
>            if (intervals[i][0] < intervals[i-1][1]) {
>                return false;
>            }
>        }
>
>        return true;
>    }
>}
>```

>[!example]- Python
>```python
>def can_attend_all_meetings(intervals):
>    # Sort by start time
>    intervals.sort(key=lambda x: x[0])
>
>    for i in range(1, len(intervals)):
>        # Check for overlap with the previous meeting
>        if intervals[i][0] < intervals[i-1][1]:
>            return False
>
>    return True
>```

>[!example]- JavaScript
>```javascript
>/**
> * @param {number[][]} intervals
> * @return {boolean}
> */
>function canAttendMeetings(intervals) {
>    // Sort by start time
>    intervals.sort((a, b) => a[0] - b[0]);
>
>    for (let i = 1; i < intervals.length; i++) {
>        // Check for overlap with the previous meeting
>        if (intervals[i][0] < intervals[i-1][1]) {
>            return false;
>        }
>    }
>
>    return true;
>}
>```

This is an O(n log n) solution, with the sort being the dominant step.

---

### Meeting Rooms II (LeetCode #253)
**Problem**: Given an array of meeting time intervals, find the minimum number of conference rooms required to hold all the meetings.

**Insight**: This is a resource allocation problem. The number of rooms required is equal to the **maximum number of meetings happening simultaneously** at any single point in time.

**Algorithm (Using a Min-Heap)**: This is a very common and efficient approach.
1.  **Sort**: Sort the intervals by their **start times**.
2.  **Initialize a Min-Heap**: The min-heap will store the `end` times of all the meetings currently in progress. The heap's size represents the number of rooms currently occupied. The `end` time at the top of the heap is the meeting that will finish the soonest.
3.  **Iterate and Allocate**:
    - Iterate through the sorted meetings.
    - For each `current_meeting`:
      - Look at the room that will be free earliest. This is the `end` time at the top of the min-heap.
      - If the heap is not empty and the `current_meeting` starts *after* or *at the same time* as the earliest-ending meeting finishes (`current_meeting.start >= heap[0]`), it means we can reuse that room. We "free up" the room by popping from the heap.
      - We then allocate a room for the `current_meeting` by pushing its `end` time onto the heap. This might be the room we just freed, or it might be a new room if we couldn't reuse one.
4.  The answer is the peak size the heap ever reached during this process. Or, more simply, the final size of the heap after all meetings have been processed, because the heap tracks all "active" rooms.

#### Implementation

>[!example]- C++
>```cpp
>class Solution {
>public:
>    int minMeetingRooms(vector<vector<int>>& intervals) {
>        if (intervals.empty()) {
>            return 0;
>        }
>
>        // Sort by start time
>        sort(intervals.begin(), intervals.end(),
>             [](const vector<int>& a, const vector<int>& b) {
>                 return a[0] < b[0];
>             });
>
>        // Min-heap to store the end times of ongoing meetings
>        priority_queue<int, vector<int>, greater<int>> occupiedRooms;
>
>        // Add the end time of the first meeting
>        occupiedRooms.push(intervals[0][1]);
>
>        for (int i = 1; i < intervals.size(); i++) {
>            vector<int> currentMeeting = intervals[i];
>
>            // If the current meeting starts after the earliest-ending meeting has finished
>            if (!occupiedRooms.empty() && currentMeeting[0] >= occupiedRooms.top()) {
>                // We can reuse this room, so "free" it by popping
>                occupiedRooms.pop();
>            }
>
>            // Allocate a room for the current meeting by adding its end time
>            occupiedRooms.push(currentMeeting[1]);
>        }
>
>        // The size of the heap at the end is the max number of rooms needed
>        return occupiedRooms.size();
>    }
>};
>```

>[!example]- Java
>```java
>class Solution {
>    public int minMeetingRooms(int[][] intervals) {
>        if (intervals == null || intervals.length == 0) {
>            return 0;
>        }
>
>        // Sort by start time
>        Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]));
>
>        // Min-heap to store the end times of ongoing meetings
>        PriorityQueue<Integer> occupiedRooms = new PriorityQueue<>();
>
>        // Add the end time of the first meeting
>        occupiedRooms.offer(intervals[0][1]);
>
>        for (int i = 1; i < intervals.length; i++) {
>            int[] currentMeeting = intervals[i];
>
>            // If the current meeting starts after the earliest-ending meeting has finished
>            if (!occupiedRooms.isEmpty() && currentMeeting[0] >= occupiedRooms.peek()) {
>                // We can reuse this room, so "free" it by polling
>                occupiedRooms.poll();
>            }
>
>            // Allocate a room for the current meeting by adding its end time
>            occupiedRooms.offer(currentMeeting[1]);
>        }
>
>        // The size of the heap at the end is the max number of rooms needed
>        return occupiedRooms.size();
>    }
>}
>```

>[!example]- Python
>```python
>import heapq
>
>def min_meeting_rooms(intervals):
>    if not intervals:
>        return 0
>
>    # Sort by start time
>    intervals.sort(key=lambda x: x[0])
>
>    # Min-heap to store the end times of ongoing meetings
>    occupied_rooms = []
>
>    # Add the end time of the first meeting
>    heapq.heappush(occupied_rooms, intervals[0][1])
>
>    for i in range(1, len(intervals)):
>        current_meeting = intervals[i]
>
>        # If the current meeting starts after the earliest-ending meeting has finished
>        if occupied_rooms and current_meeting[0] >= occupied_rooms[0]:
>            # We can reuse this room, so "free" it by popping
>            heapq.heappop(occupied_rooms)
>
>        # Allocate a room for the current meeting by adding its end time
>        heapq.heappush(occupied_rooms, current_meeting[1])
>
>    # The size of the heap at the end is the max number of rooms needed
>    return len(occupied_rooms)
>```

>[!example]- JavaScript
>```javascript
>/**
> * @param {number[][]} intervals
> * @return {number}
> */
>function minMeetingRooms(intervals) {
>    if (!intervals || intervals.length === 0) {
>        return 0;
>    }
>
>    // Sort by start time
>    intervals.sort((a, b) => a[0] - b[0]);
>
>    // Min-heap to store the end times of ongoing meetings
>    // JavaScript doesn't have a built-in heap, so we'll use an array
>    // and maintain the heap property manually
>    const occupiedRooms = [];
>
>    // Helper functions for min-heap operations
>    const heapPush = (heap, val) => {
>        heap.push(val);
>        let i = heap.length - 1;
>        while (i > 0) {
>            const parent = Math.floor((i - 1) / 2);
>            if (heap[parent] <= heap[i]) break;
>            [heap[parent], heap[i]] = [heap[i], heap[parent]];
>            i = parent;
>        }
>    };
>
>    const heapPop = (heap) => {
>        if (heap.length === 0) return null;
>        if (heap.length === 1) return heap.pop();
>
>        const min = heap[0];
>        heap[0] = heap.pop();
>        let i = 0;
>
>        while (true) {
>            let smallest = i;
>            const left = 2 * i + 1;
>            const right = 2 * i + 2;
>
>            if (left < heap.length && heap[left] < heap[smallest]) {
>                smallest = left;
>            }
>            if (right < heap.length && heap[right] < heap[smallest]) {
>                smallest = right;
>            }
>            if (smallest === i) break;
>
>            [heap[i], heap[smallest]] = [heap[smallest], heap[i]];
>            i = smallest;
>        }
>
>        return min;
>    };
>
>    // Add the end time of the first meeting
>    heapPush(occupiedRooms, intervals[0][1]);
>
>    for (let i = 1; i < intervals.length; i++) {
>        const currentMeeting = intervals[i];
>
>        // If the current meeting starts after the earliest-ending meeting has finished
>        if (occupiedRooms.length > 0 && currentMeeting[0] >= occupiedRooms[0]) {
>            // We can reuse this room, so "free" it by popping
>            heapPop(occupiedRooms);
>        }
>
>        // Allocate a room for the current meeting by adding its end time
>        heapPush(occupiedRooms, currentMeeting[1]);
>    }
>
>    // The size of the heap at the end is the max number of rooms needed
>    return occupiedRooms.length;
>}
>```

This is an O(n log n) solution. The sorting takes O(n log n), and each of the `n` heap operations (push/pop) takes O(log n).
