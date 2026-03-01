## The Difference Array Technique

The Difference Array is a clever technique that allows for efficient handling of multiple **range update** operations on an array. For problems where you need to add a value to or subtract a value from a subarray multiple times, a difference array can reduce the complexity from O(n*k) to O(n+k), where `n` is the size of the array and `k` is the number of updates.

### The Problem
Imagine you have an array, and you are given `k` update operations. Each operation tells you to add a value `v` to all elements in the range `[start, end]`. After all `k` operations are done, you need to return the final state of the array.

- **Naive Approach**: For each of the `k` updates, loop from `start` to `end` and add `v` to each element. This takes O(n) for each update, leading to a total time complexity of O(n * k), which is too slow if `n` and `k` are large.

### The Core Idea
Instead of applying the updates to the array directly, we use a "difference array" (`diff`) to store only the *changes* at the boundaries of the update ranges.

- `diff[i]` will store the change in value between the original `nums[i]` and `nums[i-1]`.
- The key insight is that an update over a range `[start, end]` only directly changes the difference at two points: `diff[start]` and `diff[end+1]`.

### The Algorithm

1.  **Initialization**: Given an original array `nums`, you can create its difference array `diff`.
    - `diff[0] = nums[0]`
    - `diff[i] = nums[i] - nums[i-1]` for `i > 0`.
    - However, it's often easier to start with an array of zeros and apply the updates.

2.  **Applying a Range Update**: To add a value `v` to all elements in the range `[start, end]`:
    a. **Mark the start**: Add `v` to `diff[start]`. This ensures that when we reconstruct the array, `v` will be added to all elements from `start` onwards.
       `diff[start] += v`
    b. **Cancel the change**: To stop the update from affecting elements beyond `end`, we must subtract `v` at the position immediately after the range.
       `if end + 1 < n:`
       `    diff[end + 1] -= v`
    This is the core of the technique, and each range update takes only **O(1)** time.

3.  **Reconstruction**: After all `k` updates have been applied to the `diff` array, you can reconstruct the final array by taking the prefix sum of the `diff` array.
    - `result[0] = diff[0]`
    - `result[i] = result[i-1] + diff[i]` for `i > 0`.
    This final step takes **O(n)** time.

### Example: Car Pooling (LeetCode #1094)
This is a classic problem that can be modeled with a difference array.
**Problem**: You are given a list of trips `[[num_passengers, start_location, end_location]]` and a car `capacity`. Determine if you can pick up and drop off all passengers for all trips.

**Difference Array Solution**:
- The "array" represents the number of passengers in the car at each location on a timeline.
- Each trip is a range update: add `num_passengers` to the range `[start_location, end_location - 1]`. (The problem states `end_location` is exclusive).
- Create a difference array (or "timeline" array) of size 1001 (since locations are 0-1000).
- For each trip: `timeline[start_location] += num_passengers` and `timeline[end_location] -= num_passengers`.
- After processing all trips, iterate through the timeline, calculating the prefix sum (the current number of passengers in the car). If at any point the number of passengers exceeds `capacity`, return `False`.

#### Implementation

>[!example]- C++
>```cpp
>class Solution {
>public:
>    bool carPooling(vector<vector<int>>& trips, int capacity) {
>        // The timeline can go from 0 to 1000.
>        vector<int> timeline(1001, 0);
>
>        // Apply all the range updates to the difference array
>        for (const auto& trip : trips) {
>            int numPassengers = trip[0];
>            int start = trip[1];
>            int end = trip[2];
>
>            timeline[start] += numPassengers;
>            timeline[end] -= numPassengers;
>        }
>
>        // Reconstruct the array by taking the prefix sum
>        // and check capacity at each step.
>        int currentPassengers = 0;
>        for (int passengersChange : timeline) {
>            currentPassengers += passengersChange;
>            if (currentPassengers > capacity) {
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
>    public boolean carPooling(int[][] trips, int capacity) {
>        // The timeline can go from 0 to 1000.
>        int[] timeline = new int[1001];
>
>        // Apply all the range updates to the difference array
>        for (int[] trip : trips) {
>            int numPassengers = trip[0];
>            int start = trip[1];
>            int end = trip[2];
>
>            timeline[start] += numPassengers;
>            timeline[end] -= numPassengers;
>        }
>
>        // Reconstruct the array by taking the prefix sum
>        // and check capacity at each step.
>        int currentPassengers = 0;
>        for (int passengersChange : timeline) {
>            currentPassengers += passengersChange;
>            if (currentPassengers > capacity) {
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
>def car_pooling(trips, capacity):
>    # The timeline can go from 0 to 1000.
>    timeline = [0] * 1001
>
>    # Apply all the range updates to the difference array
>    for num_passengers, start, end in trips:
>        timeline[start] += num_passengers
>        timeline[end] -= num_passengers
>
>    # Reconstruct the array by taking the prefix sum
>    # and check capacity at each step.
>    current_passengers = 0
>    for passengers_change in timeline:
>        current_passengers += passengers_change
>        if current_passengers > capacity:
>            return False
>
>    return True
>```

>[!example]- JavaScript
>```javascript
>/**
> * @param {number[][]} trips
> * @param {number} capacity
> * @return {boolean}
> */
>function carPooling(trips, capacity) {
>    // The timeline can go from 0 to 1000.
>    const timeline = new Array(1001).fill(0);
>
>    // Apply all the range updates to the difference array
>    for (const [numPassengers, start, end] of trips) {
>        timeline[start] += numPassengers;
>        timeline[end] -= numPassengers;
>    }
>
>    // Reconstruct the array by taking the prefix sum
>    // and check capacity at each step.
>    let currentPassengers = 0;
>    for (const passengersChange of timeline) {
>        currentPassengers += passengersChange;
>        if (currentPassengers > capacity) {
>            return false;
>        }
>    }
>
>    return true;
>}
>```

This solution is O(n + k), where `n` is the range of locations and `k` is the number of trips, which is highly efficient.
