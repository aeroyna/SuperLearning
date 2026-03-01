## The Two Heaps Pattern

The "Two Heaps" pattern is a clever and efficient technique used to solve problems that involve partitioning a dynamic set of numbers into two halves and keeping track of their central elements. The most famous application of this pattern is finding the median of a data stream.

### Core Idea
The goal is to maintain a data structure that can provide the median in O(1) time. To do this, we divide a continuous stream of numbers into two balanced halves:
1.  A **Max-Heap** to store the **smaller half** of the numbers.
2.  A **Min-Heap** to store the **larger half** of the numbers.

By maintaining these two heaps, the median is always accessible at their roots.
- The largest number in the smaller half is at the root of the max-heap.
- The smallest number in the larger half is at the root of the min-heap.

### Balancing the Heaps
To ensure the median can be found quickly, the heaps must be kept balanced:
- The sizes of the two heaps should not differ by more than 1.
- Every number in the max-heap must be less than or equal to every number in the min-heap.

### Example: Find Median from Data Stream (LeetCode #295)

**Problem**: Design a data structure that supports the following two operations:
- `addNum(int num)` - Add a number to the data structure from a data stream.
- `findMedian()` - Return the median of all elements seen so far.

**Algorithm**:
1.  **`addNum(num)`**:
    a. Add the new number to the max-heap (representing the smaller half). By convention, we add here first.
    b. To maintain the property that everything in the max-heap is smaller than everything in the min-heap, pop the largest element from the max-heap and push it onto the min-heap.
    c. **Rebalance**: If the heaps are now unbalanced (e.g., the min-heap has more elements than the max-heap), pop the smallest element from the min-heap and push it onto the max-heap. We typically keep the max-heap equal to or one larger than the min-heap.

2.  **`findMedian()`**:
    a. If the heaps have an **unequal** number of elements, the one with more elements (our max-heap by convention) holds the median at its root.
    b. If the heaps have an **equal** number of elements, the median is the average of the two root elements.

>[!example]- C++
>```cpp
>#include <queue>
>#include <vector>
>
>class MedianFinder {
>private:
>    // Max-heap to store the smaller half
>    std::priority_queue<int> smallHalf;
>    // Min-heap to store the larger half
>    std::priority_queue<int, std::vector<int>, std::greater<int>> largeHalf;
>
>public:
>    MedianFinder() {}
>
>    void addNum(int num) {
>        // 1. Add to max-heap (small_half)
>        smallHalf.push(num);
>
>        // 2. Balance by moving the largest from small_half to large_half
>        // This ensures every number in small_half <= every number in large_half
>        if (!smallHalf.empty() && !largeHalf.empty() &&
>            smallHalf.top() > largeHalf.top()) {
>            largeHalf.push(smallHalf.top());
>            smallHalf.pop();
>        }
>
>        // 3. Rebalance sizes if needed
>        // If small_half has more than one extra element
>        if (smallHalf.size() > largeHalf.size() + 1) {
>            largeHalf.push(smallHalf.top());
>            smallHalf.pop();
>        }
>
>        // If large_half has more elements
>        if (largeHalf.size() > smallHalf.size()) {
>            smallHalf.push(largeHalf.top());
>            largeHalf.pop();
>        }
>    }
>
>    double findMedian() {
>        // If total numbers are odd, the median is the root of the larger heap
>        if (smallHalf.size() > largeHalf.size()) {
>            return smallHalf.top();
>        }
>        // If total numbers are even, the median is the average of the two roots
>        else {
>            return (smallHalf.top() + largeHalf.top()) / 2.0;
>        }
>    }
>};
>```

>[!example]- Java
>```java
>import java.util.PriorityQueue;
>import java.util.Comparator;
>
>class MedianFinder {
>    // Max-heap to store the smaller half
>    private PriorityQueue<Integer> smallHalf;
>    // Min-heap to store the larger half
>    private PriorityQueue<Integer> largeHalf;
>
>    public MedianFinder() {
>        smallHalf = new PriorityQueue<>(Comparator.reverseOrder());
>        largeHalf = new PriorityQueue<>();
>    }
>
>    public void addNum(int num) {
>        // 1. Add to max-heap (small_half)
>        smallHalf.offer(num);
>
>        // 2. Balance by moving the largest from small_half to large_half
>        // This ensures every number in small_half <= every number in large_half
>        if (!smallHalf.isEmpty() && !largeHalf.isEmpty() &&
>            smallHalf.peek() > largeHalf.peek()) {
>            largeHalf.offer(smallHalf.poll());
>        }
>
>        // 3. Rebalance sizes if needed
>        // If small_half has more than one extra element
>        if (smallHalf.size() > largeHalf.size() + 1) {
>            largeHalf.offer(smallHalf.poll());
>        }
>
>        // If large_half has more elements
>        if (largeHalf.size() > smallHalf.size()) {
>            smallHalf.offer(largeHalf.poll());
>        }
>    }
>
>    public double findMedian() {
>        // If total numbers are odd, the median is the root of the larger heap
>        if (smallHalf.size() > largeHalf.size()) {
>            return smallHalf.peek();
>        }
>        // If total numbers are even, the median is the average of the two roots
>        else {
>            return (smallHalf.peek() + largeHalf.peek()) / 2.0;
>        }
>    }
>}
>```

>[!example]- Python
>```python
>import heapq
>
>class MedianFinder:
>    def __init__(self):
>        # Python's heapq is a min-heap. We simulate a max-heap
>        # by storing negative numbers.
>        self.small_half = []  # A max-heap to store the smaller half
>        self.large_half = []  # A min-heap to store the larger half
>
>    def addNum(self, num: int) -> None:
>        # 1. Add to max-heap (small_half)
>        # We negate the number to simulate a max-heap with Python's min-heap
>        heapq.heappush(self.small_half, -num)
>
>        # 2. Balance by moving the largest from small_half to large_half
>        # This ensures every number in small_half <= every number in large_half
>        if self.small_half and self.large_half and (-self.small_half[0] > self.large_half[0]):
>            val = -heapq.heappop(self.small_half)
>            heapq.heappush(self.large_half, val)
>
>        # 3. Rebalance sizes if needed
>        # If small_half has more than one extra element
>        if len(self.small_half) > len(self.large_half) + 1:
>            val = -heapq.heappop(self.small_half)
>            heapq.heappush(self.large_half, val)
>
>        # If large_half has more elements
>        if len(self.large_half) > len(self.small_half):
>            val = heapq.heappop(self.large_half)
>            heapq.heappush(self.small_half, -val)
>
>    def findMedian(self) -> float:
>        # If total numbers are odd, the median is the root of the larger heap
>        if len(self.small_half) > len(self.large_half):
>            return -self.small_half[0]
>        # If total numbers are even, the median is the average of the two roots
>        else:
>            return (-self.small_half[0] + self.large_half[0]) / 2.0
>```

>[!example]- JavaScript
>```javascript
>class MaxHeap {
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
>        const max = this.heap[0];
>        this.heap[0] = this.heap.pop();
>        this._bubbleDown(0);
>        return max;
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
>            if (this.heap[index] <= this.heap[parentIndex]) break;
>            [this.heap[index], this.heap[parentIndex]] =
>                [this.heap[parentIndex], this.heap[index]];
>            index = parentIndex;
>        }
>    }
>
>    _bubbleDown(index) {
>        while (true) {
>            let maxIndex = index;
>            const leftChild = 2 * index + 1;
>            const rightChild = 2 * index + 2;
>
>            if (leftChild < this.heap.length &&
>                this.heap[leftChild] > this.heap[maxIndex]) {
>                maxIndex = leftChild;
>            }
>            if (rightChild < this.heap.length &&
>                this.heap[rightChild] > this.heap[maxIndex]) {
>                maxIndex = rightChild;
>            }
>            if (maxIndex === index) break;
>
>            [this.heap[index], this.heap[maxIndex]] =
>                [this.heap[maxIndex], this.heap[index]];
>            index = maxIndex;
>        }
>    }
>}
>
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
>class MedianFinder {
>    constructor() {
>        // Max-heap to store the smaller half
>        this.smallHalf = new MaxHeap();
>        // Min-heap to store the larger half
>        this.largeHalf = new MinHeap();
>    }
>
>    addNum(num) {
>        // 1. Add to max-heap (small_half)
>        this.smallHalf.push(num);
>
>        // 2. Balance by moving the largest from small_half to large_half
>        // This ensures every number in small_half <= every number in large_half
>        if (this.smallHalf.size() > 0 && this.largeHalf.size() > 0 &&
>            this.smallHalf.peek() > this.largeHalf.peek()) {
>            this.largeHalf.push(this.smallHalf.pop());
>        }
>
>        // 3. Rebalance sizes if needed
>        // If small_half has more than one extra element
>        if (this.smallHalf.size() > this.largeHalf.size() + 1) {
>            this.largeHalf.push(this.smallHalf.pop());
>        }
>
>        // If large_half has more elements
>        if (this.largeHalf.size() > this.smallHalf.size()) {
>            this.smallHalf.push(this.largeHalf.pop());
>        }
>    }
>
>    findMedian() {
>        // If total numbers are odd, the median is the root of the larger heap
>        if (this.smallHalf.size() > this.largeHalf.size()) {
>            return this.smallHalf.peek();
>        }
>        // If total numbers are even, the median is the average of the two roots
>        else {
>            return (this.smallHalf.peek() + this.largeHalf.peek()) / 2.0;
>        }
>    }
>}
>```

This elegant solution allows for O(log n) insertions and O(1) median lookups.
