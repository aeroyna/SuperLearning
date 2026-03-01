# Monotonic Stack

A monotonic stack maintains elements in sorted order (either increasing or decreasing). It's a powerful technique for problems involving "next greater/smaller element."

## Core Concept

The stack maintains elements in a specific order by popping elements that violate the monotonic property when adding new elements.

### Types

1. **Monotonic Increasing Stack**: Elements increase from bottom to top
   - Useful for finding "next smaller element"
2. **Monotonic Decreasing Stack**: Elements decrease from bottom to top
   - Useful for finding "next greater element"

## Pattern: Next Greater Element

For each element, find the next element that is greater.

>[!example]- C++
>```cpp
>#include <vector>
>#include <stack>
>
>std::vector<int> nextGreaterElement(std::vector<int>& nums) {
>    int n = nums.size();
>    std::vector<int> result(n, -1);
>    std::stack<int> stack;  // Store indices
>
>    for (int i = 0; i < n; i++) {
>        // Pop elements smaller than current
>        while (!stack.empty() && nums[stack.top()] < nums[i]) {
>            int idx = stack.top();
>            stack.pop();
>            result[idx] = nums[i];
>        }
>
>        stack.push(i);
>    }
>
>    return result;
>}
>```

>[!example]- Java
>```java
>import java.util.Stack;
>
>public int[] nextGreaterElement(int[] nums) {
>    int n = nums.length;
>    int[] result = new int[n];
>    Stack<Integer> stack = new Stack<>();  // Store indices
>
>    for (int i = 0; i < n; i++) {
>        result[i] = -1;
>    }
>
>    for (int i = 0; i < n; i++) {
>        // Pop elements smaller than current
>        while (!stack.isEmpty() && nums[stack.peek()] < nums[i]) {
>            int idx = stack.pop();
>            result[idx] = nums[i];
>        }
>
>        stack.push(i);
>    }
>
>    return result;
>}
>```

>[!example]- Python
>```python
>def nextGreaterElement(nums):
>    n = len(nums)
>    result = [-1] * n
>    stack = []  # Store indices
>
>    for i in range(n):
>        # Pop elements smaller than current
>        while stack and nums[stack[-1]] < nums[i]:
>            idx = stack.pop()
>            result[idx] = nums[i]
>
>        stack.append(i)
>
>    return result
>```

>[!example]- JavaScript
>```javascript
>function nextGreaterElement(nums) {
>    const n = nums.length;
>    const result = new Array(n).fill(-1);
>    const stack = [];  // Store indices
>
>    for (let i = 0; i < n; i++) {
>        // Pop elements smaller than current
>        while (stack.length > 0 && nums[stack[stack.length - 1]] < nums[i]) {
>            const idx = stack.pop();
>            result[idx] = nums[i];
>        }
>
>        stack.push(i);
>    }
>
>    return result;
>}
>```

### Example

```
nums = [2, 1, 2, 4, 3]

i=0: stack=[], push 0 -> stack=[0]
i=1: nums[0]=2 > nums[1]=1, push 1 -> stack=[0,1]
i=2: nums[1]=1 < nums[2]=2, pop 1, result[1]=2
     nums[0]=2 = nums[2]=2, push 2 -> stack=[0,2]
i=3: nums[2]=2 < nums[3]=4, pop 2, result[2]=4
     nums[0]=2 < nums[3]=4, pop 0, result[0]=4
     push 3 -> stack=[3]
i=4: nums[3]=4 > nums[4]=3, push 4 -> stack=[3,4]

Result: [4, 2, 4, -1, -1]
```

## Pattern: Next Smaller Element

>[!example]- C++
>```cpp
>#include <vector>
>#include <stack>
>
>std::vector<int> nextSmallerElement(std::vector<int>& nums) {
>    int n = nums.size();
>    std::vector<int> result(n, -1);
>    std::stack<int> stack;
>
>    for (int i = 0; i < n; i++) {
>        // Pop elements greater than current
>        while (!stack.empty() && nums[stack.top()] > nums[i]) {
>            int idx = stack.top();
>            stack.pop();
>            result[idx] = nums[i];
>        }
>
>        stack.push(i);
>    }
>
>    return result;
>}
>```

>[!example]- Java
>```java
>import java.util.Stack;
>
>public int[] nextSmallerElement(int[] nums) {
>    int n = nums.length;
>    int[] result = new int[n];
>    Stack<Integer> stack = new Stack<>();
>
>    for (int i = 0; i < n; i++) {
>        result[i] = -1;
>    }
>
>    for (int i = 0; i < n; i++) {
>        // Pop elements greater than current
>        while (!stack.isEmpty() && nums[stack.peek()] > nums[i]) {
>            int idx = stack.pop();
>            result[idx] = nums[i];
>        }
>
>        stack.push(i);
>    }
>
>    return result;
>}
>```

>[!example]- Python
>```python
>def nextSmallerElement(nums):
>    n = len(nums)
>    result = [-1] * n
>    stack = []
>
>    for i in range(n):
>        # Pop elements greater than current
>        while stack and nums[stack[-1]] > nums[i]:
>            idx = stack.pop()
>            result[idx] = nums[i]
>
>        stack.append(i)
>
>    return result
>```

>[!example]- JavaScript
>```javascript
>function nextSmallerElement(nums) {
>    const n = nums.length;
>    const result = new Array(n).fill(-1);
>    const stack = [];
>
>    for (let i = 0; i < n; i++) {
>        // Pop elements greater than current
>        while (stack.length > 0 && nums[stack[stack.length - 1]] > nums[i]) {
>            const idx = stack.pop();
>            result[idx] = nums[i];
>        }
>
>        stack.push(i);
>    }
>
>    return result;
>}
>```

## Pattern: Previous Greater Element

Process from right to left, or look at what's in stack when pushing.

>[!example]- C++
>```cpp
>#include <vector>
>#include <stack>
>
>std::vector<int> previousGreaterElement(std::vector<int>& nums) {
>    int n = nums.size();
>    std::vector<int> result(n, -1);
>    std::stack<int> stack;
>
>    for (int i = 0; i < n; i++) {
>        while (!stack.empty() && nums[stack.top()] <= nums[i]) {
>            stack.pop();
>        }
>
>        if (!stack.empty()) {
>            result[i] = nums[stack.top()];
>        }
>
>        stack.push(i);
>    }
>
>    return result;
>}
>```

>[!example]- Java
>```java
>import java.util.Stack;
>
>public int[] previousGreaterElement(int[] nums) {
>    int n = nums.length;
>    int[] result = new int[n];
>    Stack<Integer> stack = new Stack<>();
>
>    for (int i = 0; i < n; i++) {
>        result[i] = -1;
>    }
>
>    for (int i = 0; i < n; i++) {
>        while (!stack.isEmpty() && nums[stack.peek()] <= nums[i]) {
>            stack.pop();
>        }
>
>        if (!stack.isEmpty()) {
>            result[i] = nums[stack.peek()];
>        }
>
>        stack.push(i);
>    }
>
>    return result;
>}
>```

>[!example]- Python
>```python
>def previousGreaterElement(nums):
>    n = len(nums)
>    result = [-1] * n
>    stack = []
>
>    for i in range(n):
>        while stack and nums[stack[-1]] <= nums[i]:
>            stack.pop()
>
>        if stack:
>            result[i] = nums[stack[-1]]
>
>        stack.append(i)
>
>    return result
>```

>[!example]- JavaScript
>```javascript
>function previousGreaterElement(nums) {
>    const n = nums.length;
>    const result = new Array(n).fill(-1);
>    const stack = [];
>
>    for (let i = 0; i < n; i++) {
>        while (stack.length > 0 && nums[stack[stack.length - 1]] <= nums[i]) {
>            stack.pop();
>        }
>
>        if (stack.length > 0) {
>            result[i] = nums[stack[stack.length - 1]];
>        }
>
>        stack.push(i);
>    }
>
>    return result;
>}
>```

## Classic Problems

### Daily Temperatures

Find how many days until a warmer temperature.

>[!example]- C++
>```cpp
>#include <vector>
>#include <stack>
>
>std::vector<int> dailyTemperatures(std::vector<int>& temperatures) {
>    int n = temperatures.size();
>    std::vector<int> result(n, 0);
>    std::stack<int> stack;
>
>    for (int i = 0; i < n; i++) {
>        while (!stack.empty() && temperatures[stack.top()] < temperatures[i]) {
>            int idx = stack.top();
>            stack.pop();
>            result[idx] = i - idx;
>        }
>
>        stack.push(i);
>    }
>
>    return result;
>}
>```

>[!example]- Java
>```java
>import java.util.Stack;
>
>public int[] dailyTemperatures(int[] temperatures) {
>    int n = temperatures.length;
>    int[] result = new int[n];
>    Stack<Integer> stack = new Stack<>();
>
>    for (int i = 0; i < n; i++) {
>        while (!stack.isEmpty() && temperatures[stack.peek()] < temperatures[i]) {
>            int idx = stack.pop();
>            result[idx] = i - idx;
>        }
>
>        stack.push(i);
>    }
>
>    return result;
>}
>```

>[!example]- Python
>```python
>def dailyTemperatures(temperatures):
>    n = len(temperatures)
>    result = [0] * n
>    stack = []
>
>    for i in range(n):
>        while stack and temperatures[stack[-1]] < temperatures[i]:
>            idx = stack.pop()
>            result[idx] = i - idx
>
>        stack.append(i)
>
>    return result
>```

>[!example]- JavaScript
>```javascript
>function dailyTemperatures(temperatures) {
>    const n = temperatures.length;
>    const result = new Array(n).fill(0);
>    const stack = [];
>
>    for (let i = 0; i < n; i++) {
>        while (stack.length > 0 && temperatures[stack[stack.length - 1]] < temperatures[i]) {
>            const idx = stack.pop();
>            result[idx] = i - idx;
>        }
>
>        stack.push(i);
>    }
>
>    return result;
>}
>```

### Largest Rectangle in Histogram

>[!example]- C++
>```cpp
>#include <vector>
>#include <stack>
>#include <algorithm>
>
>int largestRectangleArea(std::vector<int>& heights) {
>    std::stack<int> stack;  // Store indices
>    int max_area = 0;
>    heights.push_back(0);  // Sentinel to flush stack
>
>    for (int i = 0; i < heights.size(); i++) {
>        while (!stack.empty() && heights[stack.top()] > heights[i]) {
>            int height = heights[stack.top()];
>            stack.pop();
>            int width = stack.empty() ? i : i - stack.top() - 1;
>            max_area = std::max(max_area, height * width);
>        }
>
>        stack.push(i);
>    }
>
>    return max_area;
>}
>```

>[!example]- Java
>```java
>import java.util.Stack;
>
>public int largestRectangleArea(int[] heights) {
>    Stack<Integer> stack = new Stack<>();  // Store indices
>    int maxArea = 0;
>    int[] newHeights = new int[heights.length + 1];
>    System.arraycopy(heights, 0, newHeights, 0, heights.length);
>    newHeights[heights.length] = 0;  // Sentinel to flush stack
>
>    for (int i = 0; i < newHeights.length; i++) {
>        while (!stack.isEmpty() && newHeights[stack.peek()] > newHeights[i]) {
>            int height = newHeights[stack.pop()];
>            int width = stack.isEmpty() ? i : i - stack.peek() - 1;
>            maxArea = Math.max(maxArea, height * width);
>        }
>
>        stack.push(i);
>    }
>
>    return maxArea;
>}
>```

>[!example]- Python
>```python
>def largestRectangleArea(heights):
>    stack = []  # Store indices
>    max_area = 0
>    heights.append(0)  # Sentinel to flush stack
>
>    for i, h in enumerate(heights):
>        while stack and heights[stack[-1]] > h:
>            height = heights[stack.pop()]
>            width = i if not stack else i - stack[-1] - 1
>            max_area = max(max_area, height * width)
>
>        stack.append(i)
>
>    return max_area
>```

>[!example]- JavaScript
>```javascript
>function largestRectangleArea(heights) {
>    const stack = [];  // Store indices
>    let maxArea = 0;
>    heights.push(0);  // Sentinel to flush stack
>
>    for (let i = 0; i < heights.length; i++) {
>        while (stack.length > 0 && heights[stack[stack.length - 1]] > heights[i]) {
>            const height = heights[stack.pop()];
>            const width = stack.length === 0 ? i : i - stack[stack.length - 1] - 1;
>            maxArea = Math.max(maxArea, height * width);
>        }
>
>        stack.push(i);
>    }
>
>    return maxArea;
>}
>```

### Stock Span Problem

Count consecutive days where price was less than or equal to today.

>[!example]- C++
>```cpp
>#include <stack>
>#include <utility>
>
>class StockSpanner {
>private:
>    std::stack<std::pair<int, int>> stack;  // (price, span)
>
>public:
>    StockSpanner() {}
>
>    int next(int price) {
>        int span = 1;
>
>        while (!stack.empty() && stack.top().first <= price) {
>            span += stack.top().second;
>            stack.pop();
>        }
>
>        stack.push({price, span});
>        return span;
>    }
>};
>```

>[!example]- Java
>```java
>import java.util.Stack;
>
>class StockSpanner {
>    private Stack<int[]> stack;  // [price, span]
>
>    public StockSpanner() {
>        stack = new Stack<>();
>    }
>
>    public int next(int price) {
>        int span = 1;
>
>        while (!stack.isEmpty() && stack.peek()[0] <= price) {
>            span += stack.pop()[1];
>        }
>
>        stack.push(new int[]{price, span});
>        return span;
>    }
>}
>```

>[!example]- Python
>```python
>class StockSpanner:
>    def __init__(self):
>        self.stack = []  # (price, span)
>
>    def next(self, price):
>        span = 1
>
>        while self.stack and self.stack[-1][0] <= price:
>            span += self.stack.pop()[1]
>
>        self.stack.append((price, span))
>        return span
>```

>[!example]- JavaScript
>```javascript
>class StockSpanner {
>    constructor() {
>        this.stack = [];  // [price, span]
>    }
>
>    next(price) {
>        let span = 1;
>
>        while (this.stack.length > 0 && this.stack[this.stack.length - 1][0] <= price) {
>            span += this.stack.pop()[1];
>        }
>
>        this.stack.push([price, span]);
>        return span;
>    }
>}
>```

### Sum of Subarray Minimums

>[!example]- C++
>```cpp
>#include <vector>
>#include <stack>
>
>int sumSubarrayMins(std::vector<int>& arr) {
>    const int MOD = 1e9 + 7;
>    int n = arr.size();
>
>    // For each element, find how many subarrays it's the minimum
>    std::vector<int> left(n);   // Distance to previous smaller
>    std::vector<int> right(n);  // Distance to next smaller or equal
>    std::stack<int> stack;
>
>    for (int i = 0; i < n; i++) {
>        while (!stack.empty() && arr[stack.top()] >= arr[i]) {
>            stack.pop();
>        }
>        left[i] = stack.empty() ? i + 1 : i - stack.top();
>        stack.push(i);
>    }
>
>    while (!stack.empty()) stack.pop();
>
>    for (int i = n - 1; i >= 0; i--) {
>        while (!stack.empty() && arr[stack.top()] > arr[i]) {
>            stack.pop();
>        }
>        right[i] = stack.empty() ? n - i : stack.top() - i;
>        stack.push(i);
>    }
>
>    long long result = 0;
>    for (int i = 0; i < n; i++) {
>        result = (result + (long long)arr[i] * left[i] * right[i]) % MOD;
>    }
>
>    return result;
>}
>```

>[!example]- Java
>```java
>import java.util.Stack;
>
>public int sumSubarrayMins(int[] arr) {
>    final int MOD = 1_000_000_007;
>    int n = arr.length;
>
>    // For each element, find how many subarrays it's the minimum
>    int[] left = new int[n];   // Distance to previous smaller
>    int[] right = new int[n];  // Distance to next smaller or equal
>    Stack<Integer> stack = new Stack<>();
>
>    for (int i = 0; i < n; i++) {
>        while (!stack.isEmpty() && arr[stack.peek()] >= arr[i]) {
>            stack.pop();
>        }
>        left[i] = stack.isEmpty() ? i + 1 : i - stack.peek();
>        stack.push(i);
>    }
>
>    stack.clear();
>
>    for (int i = n - 1; i >= 0; i--) {
>        while (!stack.isEmpty() && arr[stack.peek()] > arr[i]) {
>            stack.pop();
>        }
>        right[i] = stack.isEmpty() ? n - i : stack.peek() - i;
>        stack.push(i);
>    }
>
>    long result = 0;
>    for (int i = 0; i < n; i++) {
>        result = (result + (long)arr[i] * left[i] * right[i]) % MOD;
>    }
>
>    return (int)result;
>}
>```

>[!example]- Python
>```python
>def sumSubarrayMins(arr):
>    MOD = 10**9 + 7
>    n = len(arr)
>
>    # For each element, find how many subarrays it's the minimum
>    left = [0] * n   # Distance to previous smaller
>    right = [0] * n  # Distance to next smaller or equal
>    stack = []
>
>    for i in range(n):
>        while stack and arr[stack[-1]] >= arr[i]:
>            stack.pop()
>        left[i] = i - stack[-1] if stack else i + 1
>        stack.append(i)
>
>    stack = []
>    for i in range(n - 1, -1, -1):
>        while stack and arr[stack[-1]] > arr[i]:
>            stack.pop()
>        right[i] = stack[-1] - i if stack else n - i
>        stack.append(i)
>
>    result = 0
>    for i in range(n):
>        result = (result + arr[i] * left[i] * right[i]) % MOD
>
>    return result
>```

>[!example]- JavaScript
>```javascript
>function sumSubarrayMins(arr) {
>    const MOD = 1e9 + 7;
>    const n = arr.length;
>
>    // For each element, find how many subarrays it's the minimum
>    const left = new Array(n);   // Distance to previous smaller
>    const right = new Array(n);  // Distance to next smaller or equal
>    let stack = [];
>
>    for (let i = 0; i < n; i++) {
>        while (stack.length > 0 && arr[stack[stack.length - 1]] >= arr[i]) {
>            stack.pop();
>        }
>        left[i] = stack.length === 0 ? i + 1 : i - stack[stack.length - 1];
>        stack.push(i);
>    }
>
>    stack = [];
>    for (let i = n - 1; i >= 0; i--) {
>        while (stack.length > 0 && arr[stack[stack.length - 1]] > arr[i]) {
>            stack.pop();
>        }
>        right[i] = stack.length === 0 ? n - i : stack[stack.length - 1] - i;
>        stack.push(i);
>    }
>
>    let result = 0;
>    for (let i = 0; i < n; i++) {
>        result = (result + arr[i] * left[i] * right[i]) % MOD;
>    }
>
>    return result;
>}
>```

## Template Summary

### Next Greater Element (decreasing stack)

>[!example]- C++
>```cpp
>#include <vector>
>#include <stack>
>
>std::vector<int> nextGreater(std::vector<int>& nums) {
>    std::vector<int> result(nums.size(), -1);
>    std::stack<int> stack;
>    for (int i = 0; i < nums.size(); i++) {
>        while (!stack.empty() && nums[stack.top()] < nums[i]) {
>            result[stack.top()] = nums[i];
>            stack.pop();
>        }
>        stack.push(i);
>    }
>    return result;
>}
>```

>[!example]- Java
>```java
>import java.util.Stack;
>
>public int[] nextGreater(int[] nums) {
>    int[] result = new int[nums.length];
>    Stack<Integer> stack = new Stack<>();
>    for (int i = 0; i < nums.length; i++) {
>        result[i] = -1;
>    }
>    for (int i = 0; i < nums.length; i++) {
>        while (!stack.isEmpty() && nums[stack.peek()] < nums[i]) {
>            result[stack.pop()] = nums[i];
>        }
>        stack.push(i);
>    }
>    return result;
>}
>```

>[!example]- Python
>```python
># Next Greater Element (decreasing stack)
>def nextGreater(nums):
>    result = [-1] * len(nums)
>    stack = []
>    for i in range(len(nums)):
>        while stack and nums[stack[-1]] < nums[i]:
>            result[stack.pop()] = nums[i]
>        stack.append(i)
>    return result
>```

>[!example]- JavaScript
>```javascript
>function nextGreater(nums) {
>    const result = new Array(nums.length).fill(-1);
>    const stack = [];
>    for (let i = 0; i < nums.length; i++) {
>        while (stack.length > 0 && nums[stack[stack.length - 1]] < nums[i]) {
>            result[stack.pop()] = nums[i];
>        }
>        stack.push(i);
>    }
>    return result;
>}
>```

### Next Smaller Element (increasing stack)

>[!example]- C++
>```cpp
>#include <vector>
>#include <stack>
>
>std::vector<int> nextSmaller(std::vector<int>& nums) {
>    std::vector<int> result(nums.size(), -1);
>    std::stack<int> stack;
>    for (int i = 0; i < nums.size(); i++) {
>        while (!stack.empty() && nums[stack.top()] > nums[i]) {
>            result[stack.top()] = nums[i];
>            stack.pop();
>        }
>        stack.push(i);
>    }
>    return result;
>}
>```

>[!example]- Java
>```java
>import java.util.Stack;
>
>public int[] nextSmaller(int[] nums) {
>    int[] result = new int[nums.length];
>    Stack<Integer> stack = new Stack<>();
>    for (int i = 0; i < nums.length; i++) {
>        result[i] = -1;
>    }
>    for (int i = 0; i < nums.length; i++) {
>        while (!stack.isEmpty() && nums[stack.peek()] > nums[i]) {
>            result[stack.pop()] = nums[i];
>        }
>        stack.push(i);
>    }
>    return result;
>}
>```

>[!example]- Python
>```python
># Next Smaller Element (increasing stack)
>def nextSmaller(nums):
>    result = [-1] * len(nums)
>    stack = []
>    for i in range(len(nums)):
>        while stack and nums[stack[-1]] > nums[i]:
>            result[stack.pop()] = nums[i]
>        stack.append(i)
>    return result
>```

>[!example]- JavaScript
>```javascript
>function nextSmaller(nums) {
>    const result = new Array(nums.length).fill(-1);
>    const stack = [];
>    for (let i = 0; i < nums.length; i++) {
>        while (stack.length > 0 && nums[stack[stack.length - 1]] > nums[i]) {
>            result[stack.pop()] = nums[i];
>        }
>        stack.push(i);
>    }
>    return result;
>}
>```

## Complexity

- **Time**: O(n) - Each element pushed and popped at most once
- **Space**: O(n) - Stack size

## Practice Problems

| Problem | Pattern | Difficulty |
|---------|---------|------------|
| Next Greater Element I | Basic | Easy |
| Daily Temperatures | Next Greater | Medium |
| Online Stock Span | Previous Greater | Medium |
| Largest Rectangle in Histogram | Both Sides | Hard |
| Maximal Rectangle | 2D + Histogram | Hard |
| Sum of Subarray Minimums | Count Contribution | Medium |
| Trapping Rain Water | Both Sides | Hard |
