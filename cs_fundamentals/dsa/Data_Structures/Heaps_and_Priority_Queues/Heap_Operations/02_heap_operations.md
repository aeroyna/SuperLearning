## Heap Operations in Practice

In interviews, you will use your language's standard library to handle heap operations. It's crucial to know the specific syntax for these operations.

### Basic Heap Operations

Each language provides built-in heap/priority queue implementations with similar operations but different syntax:

>[!example]- C++
>```cpp
>#include <queue>
>#include <vector>
>#include <iostream>
>
>// Max-Heap by default
>std::priority_queue<int> maxHeap;
>maxHeap.push(4);
>maxHeap.push(1);
>maxHeap.push(7);
>int largest = maxHeap.top(); // 7
>maxHeap.pop();
>
>// To create a Min-Heap, provide a custom comparator
>std::priority_queue<int, std::vector<int>, std::greater<int>> minHeap;
>minHeap.push(4);
>minHeap.push(1);
>minHeap.push(7);
>int smallest = minHeap.top(); // 1
>minHeap.pop();
>
>// C++ Operations:
>// - push(item): Pushes an item. O(log n)
>// - pop(): Removes the top element. Note: does not return the value. O(log n)
>// - top(): Returns a reference to the top element. O(1)
>// - size(): Returns the number of elements. O(1)
>// - empty(): Checks if the heap is empty. O(1)
>```

>[!example]- Java
>```java
>import java.util.PriorityQueue;
>import java.util.Comparator;
>
>// Min-Heap by default
>PriorityQueue<Integer> minHeap = new PriorityQueue<>();
>minHeap.add(4);
>minHeap.add(1);
>minHeap.add(7);
>int smallest = minHeap.peek(); // 1
>minHeap.poll(); // returns 1
>
>// To create a Max-Heap, provide a reverse order comparator
>PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
>maxHeap.add(4);
>maxHeap.add(1);
>maxHeap.add(7);
>int largest = maxHeap.peek(); // 7
>maxHeap.poll(); // returns 7
>
>// Java Operations:
>// - add(item) or offer(item): Inserts an item. O(log n)
>// - remove() or poll(): Retrieves and removes the head. O(log n)
>// - element() or peek(): Retrieves but does not remove the head. O(1)
>// - size(): Returns the number of elements. O(1)
>// - isEmpty(): Checks if the queue is empty. O(1)
>```

>[!example]- Python
>```python
>import heapq
>
># Initialize an empty list to use as a heap
>min_heap = []
>
># Push items
>heapq.heappush(min_heap, 4)
>heapq.heappush(min_heap, 1)
>heapq.heappush(min_heap, 7)
># min_heap is now [1, 4, 7], but the internal order may vary.
># Only heap[0] is guaranteed to be the smallest.
>
># Peek at smallest
>smallest = min_heap[0]  # smallest is 1
>
># Pop smallest
>popped = heapq.heappop(min_heap) # popped is 1
>
># --- Simulating a Max-Heap ---
># A common trick is to push the negative of the numbers
>max_heap = []
>heapq.heappush(max_heap, -4)
>heapq.heappush(max_heap, -1)
>heapq.heappush(max_heap, -7)
>
># The largest number is the negative of the "smallest" in the max_heap
>largest = -max_heap[0] # largest is 7
>
># Python heapq Operations:
># - heappush(heap, item): Pushes an item onto the heap. O(log n)
># - heappop(heap): Pops and returns the smallest item. O(log n)
># - heap[0]: Peeks at the smallest item without popping. O(1)
># - heapify(list): Transforms a list into a heap, in-place. O(n)
># - len(heap): Returns the number of elements. O(1)
>```

>[!example]- JavaScript
>```javascript
>// JavaScript doesn't have a built-in heap, but we can use a library
>// or implement a simple MinHeap/MaxHeap class
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
>
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
>// Usage example
>const minHeap = new MinHeap();
>minHeap.push(4);
>minHeap.push(1);
>minHeap.push(7);
>const smallest = minHeap.peek(); // 1
>const popped = minHeap.pop(); // 1
>
>// For Max-Heap, negate values or modify comparison logic
>const maxHeap = new MinHeap(); // We can reuse MinHeap with negative values
>maxHeap.push(-4);
>maxHeap.push(-1);
>maxHeap.push(-7);
>const largest = -maxHeap.peek(); // 7
>```

### Key Takeaways

- **Python**: Uses `heapq` module with functions on a list. Min-heap only. Use negation for max-heap.
- **Java**: Uses `PriorityQueue` class. Min-heap by default. Use `Comparator.reverseOrder()` for max-heap.
- **C++**: Uses `std::priority_queue`. Max-heap by default. Use `std::greater<int>` for min-heap.
- **JavaScript**: No built-in heap. Either implement your own or use a library (e.g., `heap-js`).
