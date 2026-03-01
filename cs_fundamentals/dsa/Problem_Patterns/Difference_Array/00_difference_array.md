# Difference Array

The difference array technique efficiently handles multiple range update operations. Instead of updating each element in a range (O(n) per update), we mark only the boundaries, then reconstruct the final array with a single pass.

## Overview

**Problem**: Given an array, perform multiple operations "add value v to all elements in range [l, r]"

**Naive approach**: O(n) per operation, O(q*n) total for q operations

**Difference array**: O(1) per operation, O(n) to reconstruct

## Topics

- [22.1 Difference Array Technique](01_difference_array_technique.md)
- [22.2 Practice Problems](Practice_Problems/00_practice_problems.md)

## Core Concept

For range update [l, r] += val:
- diff[l] += val (effect starts)
- diff[r+1] -= val (effect ends)

Final array = prefix sum of difference array

**Why it works**:
```
Original: [0, 0, 0, 0, 0]
Add 2 to [1, 3]:

Diff array: [0, +2, 0, 0, -2, 0]
              ↑               ↑
           start            end+1

Prefix sum: [0, 2, 2, 2, 0, 0]
```

## Implementation

>[!example]- C++
>```cpp
>vector<int> rangeAddition(int length, vector<vector<int>>& updates) {
>    vector<int> diff(length + 1, 0);
>
>    // Apply all updates to difference array
>    for (const auto& update : updates) {
>        int start = update[0];
>        int end = update[1];
>        int val = update[2];
>        diff[start] += val;
>        if (end + 1 < diff.size()) diff[end + 1] -= val;
>    }
>
>    // Reconstruct array from prefix sum
>    vector<int> result(length);
>    int current = 0;
>    for (int i = 0; i < length; i++) {
>        current += diff[i];
>        result[i] = current;
>    }
>    return result;
>}
>```

>[!example]- Java
>```java
>public int[] rangeAddition(int length, int[][] updates) {
>    int[] diff = new int[length + 1];
>
>    // Apply all updates to difference array
>    for (int[] update : updates) {
>        int start = update[0];
>        int end = update[1];
>        int val = update[2];
>        diff[start] += val;
>        if (end + 1 < diff.length) diff[end + 1] -= val;
>    }
>
>    // Reconstruct array from prefix sum
>    int[] result = new int[length];
>    int current = 0;
>    for (int i = 0; i < length; i++) {
>        current += diff[i];
>        result[i] = current;
>    }
>    return result;
>}
>```

>[!example]- Python
>```python
>def range_addition(length, updates):
>    """
>    Apply multiple range additions efficiently.
>
>    length: size of array
>    updates: list of [start, end, value] to add value to range [start, end]
>    Returns: final array after all updates
>    """
>    diff = [0] * (length + 1)
>
>    # Apply all updates to difference array
>    for start, end, val in updates:
>        diff[start] += val
>        diff[end + 1] -= val
>
>    # Reconstruct array from prefix sum
>    result = []
>    current = 0
>    for i in range(length):
>        current += diff[i]
>        result.append(current)
>
>    return result
>```

>[!example]- JavaScript
>```javascript
>function rangeAddition(length, updates) {
>    const diff = new Array(length + 1).fill(0);
>
>    // Apply all updates to difference array
>    for (const [start, end, val] of updates) {
>        diff[start] += val;
>        if (end + 1 < diff.length) diff[end + 1] -= val;
>    }
>
>    // Reconstruct array from prefix sum
>    const result = new Array(length);
>    let current = 0;
>    for (let i = 0; i < length; i++) {
>        current += diff[i];
>        result[i] = current;
>    }
>    return result;
>}
>```

## Detailed Example

```
Array size: 5, initial: [0, 0, 0, 0, 0]
Updates:
  1. Add 2 to [1, 3]
  2. Add 3 to [2, 4]
  3. Add -2 to [0, 2]

Step-by-step difference array:

Initial diff: [0, 0, 0, 0, 0, 0]

Update 1: [1,3] += 2
diff: [0, +2, 0, 0, -2, 0]

Update 2: [2,4] += 3
diff: [0, +2, +3, 0, -2, -3]

Update 3: [0,2] += -2
diff: [-2, +2, +3, +2, -2, -3]

Prefix sum:
Index 0: -2
Index 1: -2 + 2 = 0
Index 2: 0 + 3 = 3
Index 3: 3 + 2 = 5
Index 4: 5 + (-2) = 3

Result: [-2, 0, 3, 5, 3]

Verification:
Index 0: Update 3 applies → -2 ✓
Index 1: Updates 1, 3 → 2 + (-2) = 0 ✓
Index 2: Updates 1, 2, 3 → 2 + 3 + (-2) = 3 ✓
Index 3: Updates 1, 2 → 2 + 3 = 5 ✓
Index 4: Update 2 → 3 ✓
```

## Common Applications

### Range Addition

LeetCode 370: Range Addition

```python
def getModifiedArray(length, updates):
    diff = [0] * (length + 1)
    for start, end, val in updates:
        diff[start] += val
        diff[end + 1] -= val

    result = [0] * length
    prefix = 0
    for i in range(length):
        prefix += diff[i]
        result[i] = prefix

    return result
```

### Corporate Flight Bookings

```python
def corpFlightBookings(bookings, n):
    """
    bookings[i] = [first, last, seats]
    Return array where result[i] = total seats booked for flight i
    """
    diff = [0] * (n + 1)

    for first, last, seats in bookings:
        diff[first - 1] += seats  # 1-indexed to 0-indexed
        diff[last] -= seats

    result = []
    total = 0
    for i in range(n):
        total += diff[i]
        result.append(total)

    return result
```

### Car Pooling

```python
def carPooling(trips, capacity):
    """
    trips[i] = [numPassengers, from, to]
    Return True if all trips can be completed with capacity
    """
    # Find maximum location
    max_loc = max(trip[2] for trip in trips)
    diff = [0] * (max_loc + 1)

    for passengers, start, end in trips:
        diff[start] += passengers
        diff[end] -= passengers  # Passengers leave at 'to'

    current = 0
    for i in range(max_loc + 1):
        current += diff[i]
        if current > capacity:
            return False

    return True
```

## 2D Difference Array

For 2D range updates (add value to rectangle):

```python
def range_addition_2d(n, m, updates):
    """
    updates: [(r1, c1, r2, c2, val), ...]
    Add val to rectangle from (r1,c1) to (r2,c2)
    """
    diff = [[0] * (m + 2) for _ in range(n + 2)]

    for r1, c1, r2, c2, val in updates:
        diff[r1][c1] += val
        diff[r1][c2 + 1] -= val
        diff[r2 + 1][c1] -= val
        diff[r2 + 1][c2 + 1] += val

    # Reconstruct with 2D prefix sum
    result = [[0] * m for _ in range(n)]
    for i in range(n):
        for j in range(m):
            diff[i][j] += (diff[i-1][j] if i > 0 else 0)
            diff[i][j] += (diff[i][j-1] if j > 0 else 0)
            diff[i][j] -= (diff[i-1][j-1] if i > 0 and j > 0 else 0)
            result[i][j] = diff[i][j]

    return result
```

## When to Use

**Use difference array when**:
- Multiple range updates, single final query
- Updates are additive
- Query only after all updates complete

**Don't use when**:
- Need intermediate results between updates
- Updates are not additive (e.g., set range to value)
- Single update scenario (just iterate normally)

## Complexity Analysis

| Operation | Naive | Difference Array |
|-----------|-------|------------------|
| Single update | O(n) | O(1) |
| q updates | O(q*n) | O(q) |
| Reconstruct | - | O(n) |
| Total | O(q*n) | O(q + n) |

## Common Pitfalls

1. **Off-by-one at end**: Remember `diff[r+1] -= val`, not `diff[r]`
2. **Array bounds**: Difference array needs length+1 size
3. **1-indexed vs 0-indexed**: Convert carefully
4. **Forgetting reconstruction**: Difference array isn't the answer, prefix sum is

## Key Interview Problems

| Problem | Variant | Difficulty | LeetCode Link |
| --------- | --------- | ------------ | --- |
| Range Addition | Basic | Medium | [Link](https://leetcode.com/problems/range-addition/) |
| Corporate Flight Bookings | Application | Medium | [Link](https://leetcode.com/problems/corporate-flight-bookings/) |
| Car Pooling | Capacity constraint | Medium | [Link](https://leetcode.com/problems/car-pooling/) |
| My Calendar II | Double booking | Medium | [Link](https://leetcode.com/problems/my-calendar-ii/) |


### Practice
- [Practice Problems](Practice_Problems/00_practice_problems.md)