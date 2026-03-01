## Monotonic Queue

A Monotonic Queue (specifically, a monotonic deque) is an advanced data structure that maintains its elements in a sorted order (either increasing or decreasing). Its primary use case is to find the minimum or maximum element in a sliding window in O(1) time.

### Core Idea
A monotonic queue is designed to efficiently answer the question: "What is the max/min value in the current window?"

Let's consider a **monotonically decreasing** queue, used to find the maximum in a sliding window.
- The queue will store elements (or their indices) from the current window.
- The key property: The elements in the queue will always be in decreasing order, so the front of the queue is always the largest element in the current window.

How is this property maintained?
- **When adding a new element (`x`)**: Before adding `x` to the back of the queue, you remove all elements from the back that are smaller than `x`. This ensures that `x` doesn't break the decreasing property and that any smaller elements that came before it (which can no longer be the maximum) are discarded.
- **When shrinking the window**: If the element at the front of the queue (the current max) is the same as the element that is sliding out of the window from the left, you remove it from the front of the queue.

### Example: Sliding Window Maximum (LeetCode #239)

**Problem**: You are given an array of integers `nums` and a window of size `k`. You are sliding the window from left to right. You can only see the `k` numbers in the window. Return an array of the maximum value in each window.

**Solution**: A monotonic deque is the optimal solution, providing an O(n) time complexity.

1.  Initialize an empty `deque` to store indices of the numbers in `nums`.
2.  Initialize an empty `result` array.
3.  Iterate through `nums` with index `i`:
    a. **Maintain Window Size**: If the index at the front of the deque is no longer in the current window (i.e., `deque[0] <= i - k`), pop it from the left.
    b. **Maintain Monotonicity**: While the deque is not empty and the number at the index at the back of the deque (`nums[deque[-1]]`) is less than or equal to the current number (`nums[i]`), pop from the right. This removes all smaller elements that are no longer candidates for the maximum.
    c. **Add Current Element**: Append the current index `i` to the deque.
    d. **Record Result**: If the window is fully formed (`i >= k - 1`), the maximum for the current window is the number at the index at the front of the deque (`nums[deque[0]]`). Add this to the `result` array.

```python
from collections import deque

def max_sliding_window(nums, k):
    result = []
    # Deque will store indices of elements in the current window
    q = deque()

    for i, num in enumerate(nums):
        # 1. Remove indices from the front that are out of the current window
        if q and q[0] <= i - k:
            q.popleft()

        # 2. Maintain decreasing monotonicity: remove smaller elements from the back
        while q and nums[q[-1]] <= num:
            q.pop()

        # 3. Add the current element's index to the back
        q.append(i)

        # 4. If the window is fully formed, the front of the deque is the max
        if i >= k - 1:
            result.append(nums[q[0]])

    return result
```

>[!example]- C++
>```cpp
>#include <deque>
>#include <vector>
>
>using namespace std;
>
>vector<int> maxSlidingWindow(vector<int>& nums, int k) {
>    deque<int> dq;
>    vector<int> result;
>    for (int i = 0; i < nums.size(); ++i) {
>        // 1. Remove indices from the front that are out of the current window
>        if (!dq.empty() && dq.front() == i - k) {
>            dq.pop_front();
>        }
>        // 2. Maintain decreasing monotonicity: remove smaller elements from the back
>        while (!dq.empty() && nums[dq.back()] <= nums[i]) {
>            dq.pop_back();
>        }
>        // 3. Add the current element's index to the back
>        dq.push_back(i);
>        // 4. If the window is fully formed, the front of the deque is the max
>        if (i >= k - 1) {
>            result.push_back(nums[dq.front()]);
>        }
>    }
>    return result;
>}
>```

>[!example]- Java
>```java
>import java.util.ArrayDeque;
>import java.util.Deque;
>
>public int[] maxSlidingWindow(int[] nums, int k) {
>    if (nums == null || k <= 0) return new int[0];
>    int n = nums.length;
>    int[] result = new int[n - k + 1];
>    int ri = 0;
>    Deque<Integer> dq = new ArrayDeque<>();
>    
>    for (int i = 0; i < n; i++) {
>        // 1. Remove indices from the front that are out of the current window
>        if (!dq.isEmpty() && dq.peekFirst() == i - k) {
>            dq.pollFirst();
>        }
>        // 2. Maintain decreasing monotonicity: remove smaller elements from the back
>        while (!dq.isEmpty() && nums[dq.peekLast()] <= nums[i]) {
>            dq.pollLast();
>        }
>        // 3. Add the current element's index to the back
>        dq.offerLast(i);
>        // 4. If the window is fully formed, the front of the deque is the max
>        if (i >= k - 1) {
>            result[ri++] = nums[dq.peekFirst()];
>        }
>    }
>    return result;
>}
>```

>[!example]- JavaScript
>```javascript
>var maxSlidingWindow = function(nums, k) {
>    const result = [];
>    const dq = []; // Will store indices
>    
>    for (let i = 0; i < nums.length; i++) {
>        // 1. Remove indices from the front that are out of the current window
>        if (dq.length > 0 && dq[0] === i - k) {
>            dq.shift();
>        }
>        // 2. Maintain decreasing monotonicity: remove smaller elements from the back
>        while (dq.length > 0 && nums[dq[dq.length - 1]] <= nums[i]) {
>            dq.pop();
>        }
>        // 3. Add the current element's index to the back
>        dq.push(i);
>        // 4. If the window is fully formed, the front of the deque is the max
>        if (i >= k - 1) {
>            result.push(nums[dq[0]]);
>        }
>    }
>    return result;
>};
>```
This O(n) approach is far superior to the naive O(n*k) solution of re-calculating the max for each window. Each index is added and removed from the deque at most once.
