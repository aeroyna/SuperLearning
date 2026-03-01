# Practice Problems: Difference Array

Efficient range updates.

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Range Addition](https://leetcode.com/problems/range-addition/) | Medium | `arr[start] += val`, `arr[end+1] -= val`. Prefix sum for result. |
| [Corporate Flight Bookings](https://leetcode.com/problems/corporate-flight-bookings/) | Medium | `bookings[i][0]-1` increment, `bookings[i][1]` decrement. |
| [Car Pooling](https://leetcode.com/problems/car-pooling/) | Medium | Add passengers at start, subtract at end. Check capacity. |