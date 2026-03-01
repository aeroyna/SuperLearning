# Monotonic Stack

A monotonic stack maintains elements in sorted order (increasing or decreasing), enabling efficient solutions for problems involving "next greater/smaller element" patterns. Elements that would break the monotonic property are popped before insertion.

## Overview

The monotonic stack transforms O(n²) brute-force solutions into O(n) by maintaining a stack where:
- **Monotonically Increasing**: Each element is greater than or equal to the one below
- **Monotonically Decreasing**: Each element is less than or equal to the one below

## Topics

- [5.4.1 Monotonic Stack Fundamentals](01_monotonic_stack.md)
- [5.4.2 Monotonic Queue](02_monotonic_queue.md)

## Core Mechanism

### Building Intuition

For "next greater element": as we process elements left-to-right, we maintain a stack of elements waiting for their "next greater." When we find a larger element, all smaller waiting elements have found their answer.

>[!example]- C++
>```cpp
>vector<int> nextGreaterElement(vector<int>& nums) {
>    int n = nums.size();
>    vector<int> result(n, -1);
>    stack<int> s; // Stores indices
>    
>    for (int i = 0; i < n; i++) {
>        while (!s.empty() && nums[s.top()] < nums[i]) {
>            int idx = s.top();
>            s.pop();
>            result[idx] = nums[i];
>        }
>        s.push(i);
>    }
>    return result;
>}
>```

>[!example]- Java
>```java
>public int[] nextGreaterElement(int[] nums) {
>    int n = nums.length;
>    int[] result = new int[n];
>    Arrays.fill(result, -1);
>    Stack<Integer> stack = new Stack<>(); // Stores indices
>    
>    for (int i = 0; i < n; i++) {
>        while (!stack.isEmpty() && nums[stack.peek()] < nums[i]) {
>            int idx = stack.pop();
>            result[idx] = nums[i];
>        }
>        stack.push(i);
>    }
>    return result;
>}
>```

>[!example]- Python
>```python
>def next_greater_element(nums):
>    n = len(nums)
>    result = [-1] * n
>    stack = []  # Stores indices
>
>    for i in range(n):
>        # Pop all elements smaller than current
>        while stack and nums[stack[-1]] < nums[i]:
>            idx = stack.pop()
>            result[idx] = nums[i]  # nums[i] is the next greater
>        stack.append(i)
>
>    return result
>```

>[!example]- JavaScript
>```javascript
>function nextGreaterElement(nums) {
>    const n = nums.length;
>    const result = new Array(n).fill(-1);
>    const stack = []; // Stores indices
>    
>    for (let i = 0; i < n; i++) {
>        while (stack.length > 0 && nums[stack[stack.length - 1]] < nums[i]) {
>            const idx = stack.pop();
>            result[idx] = nums[i];
>        }
>        stack.push(i);
>    }
>    return result;
>}
>```

**Stack state visualization** for `nums = [2, 1, 2, 4, 3]`:
```
i=0, nums[0]=2: stack=[] → push 0 → stack=[0]
i=1, nums[1]=1: 1<2, push 1 → stack=[0,1]
i=2, nums[2]=2: 2>1, pop 1, result[1]=2; 2<=2, push 2 → stack=[0,2]
i=3, nums[3]=4: 4>2, pop 2, result[2]=4; 4>2, pop 0, result[0]=4; push 3 → stack=[3]
i=4, nums[4]=3: 3<4, push 4 → stack=[3,4]

Result: [4, 2, 4, -1, -1]
```

## Pattern Variations

### Next Greater Element (Right)

```python
# Decreasing stack, process left-to-right
for i in range(n):
    while stack and nums[stack[-1]] < nums[i]:
        result[stack.pop()] = nums[i]
    stack.append(i)
```

### Previous Greater Element (Left)

```python
# Decreasing stack, process left-to-right
for i in range(n):
    while stack and nums[stack[-1]] <= nums[i]:
        stack.pop()
    result[i] = nums[stack[-1]] if stack else -1
    stack.append(i)
```

### Next Smaller Element

```python
# Increasing stack (opposite direction)
for i in range(n):
    while stack and nums[stack[-1]] > nums[i]:
        result[stack.pop()] = nums[i]
    stack.append(i)
```

## Decision Framework

```
Looking for what?           Stack type        Direction
---------------------------------------------------------
Next Greater (right)    →   Decreasing    →   Left to Right
Previous Greater (left) →   Decreasing    →   Left to Right
Next Smaller (right)    →   Increasing    →   Left to Right
Previous Smaller (left) →   Increasing    →   Left to Right
```

**Memory tip**: Stack is opposite of what you're looking for. Looking for greater? Use decreasing stack (so you pop smaller elements).

## Classic Applications

### Daily Temperatures

```python
def daily_temperatures(temperatures):
    n = len(temperatures)
    result = [0] * n
    stack = []

    for i in range(n):
        while stack and temperatures[stack[-1]] < temperatures[i]:
            prev_idx = stack.pop()
            result[prev_idx] = i - prev_idx
        stack.append(i)

    return result
```

### Largest Rectangle in Histogram

```python
def largest_rectangle_area(heights):
    stack = []  # Stores indices
    max_area = 0
    heights = [0] + heights + [0]  # Sentinel values

    for i, h in enumerate(heights):
        while stack and heights[stack[-1]] > h:
            height = heights[stack.pop()]
            width = i - stack[-1] - 1
            max_area = max(max_area, height * width)
        stack.append(i)

    return max_area
```

**Why this works**: For each bar, we need to find the first smaller bar on left and right. The stack naturally maintains this: when we pop, current position is the right boundary, and new stack top is the left boundary.

### Trapping Rain Water

```python
def trap(height):
    stack = []
    water = 0

    for i, h in enumerate(height):
        while stack and height[stack[-1]] < h:
            bottom = height[stack.pop()]
            if not stack:
                break
            left = stack[-1]
            width = i - left - 1
            bounded_height = min(h, height[left]) - bottom
            water += width * bounded_height
        stack.append(i)

    return water
```

## Monotonic Deque (Sliding Window Maximum)

Extend the concept to maintain maximum in a sliding window:

```python
def max_sliding_window(nums, k):
    dq = deque()  # Stores indices, values are decreasing
    result = []

    for i in range(len(nums)):
        # Remove elements outside window
        while dq and dq[0] <= i - k:
            dq.popleft()

        # Maintain decreasing order
        while dq and nums[dq[-1]] < nums[i]:
            dq.pop()

        dq.append(i)

        # Window is full, record maximum
        if i >= k - 1:
            result.append(nums[dq[0]])

    return result
```

## Complexity Analysis

| Problem Type | Brute Force | Monotonic Stack |
|--------------|-------------|-----------------|
| Next greater element | O(n²) | O(n) |
| Largest rectangle | O(n²) | O(n) |
| Sliding window max | O(n*k) | O(n) |

**Why O(n)**: Each element is pushed once and popped at most once.

## Common Pitfalls

1. **Strict vs non-strict**: Use `<` vs `<=` depending on whether equal elements should count
2. **Storing indices vs values**: Usually store indices to calculate distances
3. **Circular arrays**: Process array twice or extend with itself
4. **Empty stack check**: Always check stack before accessing `stack[-1]`

## Key Interview Problems

| Problem | Variant | Difficulty | LeetCode Link |
| --------- | --------- | ------------ | --- |
| Next Greater Element I | Basic | Easy | [Link](https://leetcode.com/problems/next-greater-element-i/) |
| Daily Temperatures | Next greater + distance | Medium | [Link](https://leetcode.com/problems/daily-temperatures/) |
| Next Greater Element II | Circular | Medium | [Link](https://leetcode.com/problems/next-greater-element-ii/) |
| Largest Rectangle in Histogram | Next smaller both sides | Hard | [Link](https://leetcode.com/problems/largest-rectangle-in-histogram/) |
| Trapping Rain Water | Stack-based approach | Hard | [Link](https://leetcode.com/problems/trapping-rain-water/) |
| Sliding Window Maximum | Monotonic deque | Hard | [Link](https://leetcode.com/problems/sliding-window-maximum/) |
