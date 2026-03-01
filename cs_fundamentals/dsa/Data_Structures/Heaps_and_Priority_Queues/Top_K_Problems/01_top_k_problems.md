## Solving "Top K" Problems with Heaps

A very common category of interview questions involves finding the "Top K" elements in a collection. For example:
- Find the Kth largest element in an array.
- Find the K most frequent elements.
- Find the K closest points to the origin.

Heaps (Priority Queues) are the perfect tool for solving these problems efficiently. The general approach has a time complexity of **O(n log k)**, which is much better than sorting the entire collection (O(n log n)).

### The Core Pattern
The key idea is to maintain a heap of size **K**.

- To find the **Kth largest** or **Top K largest** elements, you use a **min-heap**.
- To find the **Kth smallest** or **Top K smallest** elements, you use a **max-heap**.

This might seem counter-intuitive, but the logic is as follows:

### Example: Find the Kth Largest Element (LeetCode #215)
Let's find the Kth largest element using a **min-heap** of size K.

**Algorithm**:
1. Create a min-heap.
2. Iterate through the numbers in the input array.
3. For each number, push it onto the min-heap.
4. **Crucially, if the heap's size grows larger than K, pop the smallest element**. This is the key step. Popping the minimum element ensures that the heap always holds the K largest elements encountered *so far*.
5. After iterating through all the numbers, the heap contains the top K largest elements from the entire array.
6. The root of the min-heap (the smallest element among the top K) is the Kth largest element overall.

>[!example]- C++
>```cpp
>#include <queue>
>#include <vector>
>
>int findKthLargest(std::vector<int>& nums, int k) {
>    // Use a min-heap to keep track of the k largest elements
>    std::priority_queue<int, std::vector<int>, std::greater<int>> minHeap;
>
>    for (int num : nums) {
>        // Push the current number onto the heap
>        minHeap.push(num);
>
>        // If the heap size exceeds k, remove the smallest element
>        if (minHeap.size() > k) {
>            minHeap.pop();
>        }
>    }
>
>    // The root of the heap is the kth largest element
>    return minHeap.top();
>}
>
>// Example usage:
>// std::vector<int> nums = {3, 2, 1, 5, 6, 4};
>// int k = 2;
>// The 2nd largest element is 5
>// std::cout << findKthLargest(nums, k); // Output: 5
>```

>[!example]- Java
>```java
>import java.util.PriorityQueue;
>
>class Solution {
>    public int findKthLargest(int[] nums, int k) {
>        // Use a min-heap to keep track of the k largest elements
>        PriorityQueue<Integer> minHeap = new PriorityQueue<>();
>
>        for (int num : nums) {
>            // Push the current number onto the heap
>            minHeap.offer(num);
>
>            // If the heap size exceeds k, remove the smallest element
>            if (minHeap.size() > k) {
>                minHeap.poll();
>            }
>        }
>
>        // The root of the heap is the kth largest element
>        return minHeap.peek();
>    }
>}
>
>// Example usage:
>// int[] nums = {3, 2, 1, 5, 6, 4};
>// int k = 2;
>// The 2nd largest element is 5
>// System.out.println(findKthLargest(nums, k)); // Output: 5
>```

>[!example]- Python
>```python
>import heapq
>
>def find_kth_largest(nums, k):
>    # We use a min-heap to keep track of the k largest elements seen so far
>    min_heap = []
>
>    for num in nums:
>        # Push the current number onto the heap
>        heapq.heappush(min_heap, num)
>
>        # If the heap size exceeds k, remove the smallest element
>        if len(min_heap) > k:
>            heapq.heappop(min_heap)
>
>    # The root of the heap is the kth largest element
>    return min_heap[0]
>
># Example:
>nums = [3, 2, 1, 5, 6, 4]
>k = 2
># The 2nd largest element is 5
>print(find_kth_largest(nums, k)) # Output: 5
>```

>[!example]- JavaScript
>```javascript
>class MinHeap {
>    constructor() {
>        this.heap = [];
>    }
>
>    push(val) {
>        this.heap.push(val);
>        this._bubbleUp(this.heap.length - 1);
>    }
>
>    pop() {
>        if (this.heap.length === 0) return null;
>        if (this.heap.length === 1) return this.heap.pop();
>        const min = this.heap[0];
>        this.heap[0] = this.heap.pop();
>        this._bubbleDown(0);
>        return min;
>    }
>
>    peek() {
>        return this.heap.length > 0 ? this.heap[0] : null;
>    }
>
>    size() {
>        return this.heap.length;
>    }
>
>    _bubbleUp(index) {
>        while (index > 0) {
>            const parentIndex = Math.floor((index - 1) / 2);
>            if (this.heap[index] >= this.heap[parentIndex]) break;
>            [this.heap[index], this.heap[parentIndex]] =
>                [this.heap[parentIndex], this.heap[index]];
>            index = parentIndex;
>        }
>    }
>
>    _bubbleDown(index) {
>        while (true) {
>            let minIndex = index;
>            const leftChild = 2 * index + 1;
>            const rightChild = 2 * index + 2;
>
>            if (leftChild < this.heap.length &&
>                this.heap[leftChild] < this.heap[minIndex]) {
>                minIndex = leftChild;
>            }
>            if (rightChild < this.heap.length &&
>                this.heap[rightChild] < this.heap[minIndex]) {
>                minIndex = rightChild;
>            }
>            if (minIndex === index) break;
>
>            [this.heap[index], this.heap[minIndex]] =
>                [this.heap[minIndex], this.heap[index]];
>            index = minIndex;
>        }
>    }
>}
>
>function findKthLargest(nums, k) {
>    // Use a min-heap to keep track of the k largest elements
>    const minHeap = new MinHeap();
>
>    for (const num of nums) {
>        // Push the current number onto the heap
>        minHeap.push(num);
>
>        // If the heap size exceeds k, remove the smallest element
>        if (minHeap.size() > k) {
>            minHeap.pop();
>        }
>    }
>
>    // The root of the heap is the kth largest element
>    return minHeap.peek();
>}
>
>// Example:
>const nums = [3, 2, 1, 5, 6, 4];
>const k = 2;
>// The 2nd largest element is 5
>console.log(findKthLargest(nums, k)); // Output: 5
>```

### Why does this work?
The min-heap acts as a "gatekeeper" for the top K club. When a new number arrives, it's compared against the smallest member of the club (the root of the min-heap).
- If the new number is smaller, it's not worthy of being in the top K, and the heap remains unchanged (or the new number is added and then immediately popped if we are strictly maintaining size K).
- If the new number is larger, it deserves a spot. The smallest member of the club is kicked out (`heappop`), and the new, larger number takes its place (`heappush`).

This ensures that at the end, the heap holds exactly the top K elements, and its root is the Kth largest. This same pattern can be adapted for finding the K most frequent elements (where you'd store frequencies in the heap) or K closest points (storing distances in the heap).
