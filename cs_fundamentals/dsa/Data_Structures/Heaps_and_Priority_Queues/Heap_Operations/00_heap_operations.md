# Heap Operations

Understanding heap operations at the implementation level—how elements bubble up and down, how the array structure maintains the heap property—is essential for both using heaps effectively and implementing them from scratch when required.

## Overview

A binary heap is a complete binary tree stored in an array where each parent has a specific ordering relationship with its children (min-heap: parent ≤ children, max-heap: parent ≥ children).

## Topics

- [10.1.1 Heap Fundamentals](01_heap_fundamentals.md)
- [10.1.2 Heap Operations](02_heap_operations.md)

## Array Representation

```
Min-Heap:     1
            /   \
           3     5
          / \   /
         7   4 8

Array: [1, 3, 5, 7, 4, 8]
Index:  0  1  2  3  4  5

Navigation (0-indexed):
- Parent of i: (i - 1) // 2
- Left child of i: 2*i + 1
- Right child of i: 2*i + 2
```

**Why array works**: Complete binary tree has no gaps—level-order maps directly to contiguous array indices.

## Core Operations

### Insert (Push) - O(log n)

Add element at end, then "bubble up" to restore heap property.

>[!example]- C++
>```cpp
>class MinHeap {
>    vector<int> heap;
>    
>    void siftUp(int idx) {
>        while (idx > 0) {
>            int parent = (idx - 1) / 2;
>            if (heap[idx] < heap[parent]) {
>                swap(heap[idx], heap[parent]);
>                idx = parent;
>            } else {
>                break;
>            }
>        }
>    }
>    
>public:
>    void push(int val) {
>        heap.push_back(val);
>        siftUp(heap.size() - 1);
>    }
>};
>```

>[!example]- Java
>```java
>class MinHeap {
>    List<Integer> heap = new ArrayList<>();
>    
>    public void push(int val) {
>        heap.add(val);
>        siftUp(heap.size() - 1);
>    }
>    
>    private void siftUp(int idx) {
>        while (idx > 0) {
>            int parent = (idx - 1) / 2;
>            if (heap.get(idx) < heap.get(parent)) {
>                Collections.swap(heap, idx, parent);
>                idx = parent;
>            } else {
>                break;
>            }
>        }
>    }
>}
>```

>[!example]- Python
>```python
>def push(self, val):
>    self.heap.append(val)
>    self._sift_up(len(self.heap) - 1)
>
>def _sift_up(self, idx):
>    while idx > 0:
>        parent = (idx - 1) // 2
>        if self.heap[idx] < self.heap[parent]:  # Min-heap
>            self.heap[idx], self.heap[parent] = self.heap[parent], self.heap[idx]
>            idx = parent
>        else:
>            break
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
>        this.siftUp(this.heap.length - 1);
>    }
>    
>    siftUp(idx) {
>        while (idx > 0) {
>            const parent = Math.floor((idx - 1) / 2);
>            if (this.heap[idx] < this.heap[parent]) {
>                [this.heap[idx], this.heap[parent]] = [this.heap[parent], this.heap[idx]];
>                idx = parent;
>            } else {
>                break;
>            }
>        }
>    }
>}
>```

**Execution trace** (inserting 2 into min-heap [1, 3, 5, 7, 4, 8]):
```
[1, 3, 5, 7, 4, 8, 2]  ← Append 2
                   ^
[1, 3, 2, 7, 4, 8, 5]  ← Swap with parent 5 (idx 2)
       ^
[1, 3, 2, 7, 4, 8, 5]  ← 2 > 1, stop (heap restored)
```

### Extract Min/Max (Pop) - O(log n)

Remove root, move last element to root, then "sift down" to restore heap property.

>[!example]- C++
>```cpp
>int pop() {
>    if (heap.empty()) throw runtime_error("Heap empty");
>    
>    int root = heap[0];
>    int last = heap.back();
>    heap.pop_back();
>    
>    if (!heap.empty()) {
>        heap[0] = last;
>        siftDown(0);
>    }
>    return root;
>}
>
>void siftDown(int idx) {
>    int n = heap.size();
>    while (true) {
>        int smallest = idx;
>        int left = 2 * idx + 1;
>        int right = 2 * idx + 2;
>        
>        if (left < n && heap[left] < heap[smallest]) smallest = left;
>        if (right < n && heap[right] < heap[smallest]) smallest = right;
>        
>        if (smallest != idx) {
>            swap(heap[idx], heap[smallest]);
>            idx = smallest;
>        } else {
>            break;
>        }
>    }
>}
>```

>[!example]- Java
>```java
>public int pop() {
>    if (heap.isEmpty()) throw new IllegalStateException("Heap empty");
>    
>    int root = heap.get(0);
>    int last = heap.remove(heap.size() - 1);
>    
>    if (!heap.isEmpty()) {
>        heap.set(0, last);
>        siftDown(0);
>    }
>    return root;
>}
>
>private void siftDown(int idx) {
>    int n = heap.size();
>    while (true) {
>        int smallest = idx;
>        int left = 2 * idx + 1;
>        int right = 2 * idx + 2;
>        
>        if (left < n && heap.get(left) < heap.get(smallest)) smallest = left;
>        if (right < n && heap.get(right) < heap.get(smallest)) smallest = right;
>        
>        if (smallest != idx) {
>            Collections.swap(heap, idx, smallest);
>            idx = smallest;
>        } else {
>            break;
>        }
>    }
>}
>```

>[!example]- Python
>```python
>def pop(self):
>    if not self.heap:
>        raise IndexError("Heap empty")
>
>    root = self.heap[0]
>    last = self.heap.pop()
>
>    if self.heap:
>        self.heap[0] = last
>        self._sift_down(0)
>
>    return root
>
>def _sift_down(self, idx):
>    n = len(self.heap)
>    while True:
>        smallest = idx
>        left = 2 * idx + 1
>        right = 2 * idx + 2
>
>        if left < n and self.heap[left] < self.heap[smallest]:
>            smallest = left
>        if right < n and self.heap[right] < self.heap[smallest]:
>            smallest = right
>
>        if smallest != idx:
>            self.heap[idx], self.heap[smallest] = self.heap[smallest], self.heap[idx]
>            idx = smallest
>        else:
>            break
>```

>[!example]- JavaScript
>```javascript
>pop() {
>    if (this.heap.length === 0) throw new Error("Heap empty");
>    
>    const root = this.heap[0];
>    const last = this.heap.pop();
>    
>    if (this.heap.length > 0) {
>        this.heap[0] = last;
>        this.siftDown(0);
>    }
>    return root;
>}
>
>siftDown(idx) {
>    const n = this.heap.length;
>    while (true) {
>        let smallest = idx;
>        const left = 2 * idx + 1;
>        const right = 2 * idx + 2;
>        
>        if (left < n && this.heap[left] < this.heap[smallest]) smallest = left;
>        if (right < n && this.heap[right] < this.heap[smallest]) smallest = right;
>        
>        if (smallest !== idx) {
>            [this.heap[idx], this.heap[smallest]] = [this.heap[smallest], this.heap[idx]];
>            idx = smallest;
>        } else {
>            break;
>        }
>    }
>}
>```

**Why swap with last element**: Removing root directly would break the complete tree property. Swapping with last maintains completeness.

### Peek - O(1)

```python
def peek(self):
    if not self.heap:
        raise IndexError("Heap empty")
    return self.heap[0]
```

### Heapify (Build Heap) - O(n)

Convert arbitrary array to heap in O(n)—not O(n log n)!

>[!example]- C++
>```cpp
>void heapify(vector<int>& arr) {
>    heap = arr;
>    for (int i = (heap.size() - 2) / 2; i >= 0; i--) {
>        siftDown(i);
>    }
>}
>```

>[!example]- Java
>```java
>public void heapify(List<Integer> arr) {
>    heap = new ArrayList<>(arr);
>    for (int i = (heap.size() - 2) / 2; i >= 0; i--) {
>        siftDown(i);
>    }
>}
>```

>[!example]- Python
>```python
>def heapify(self, arr):
>    self.heap = arr
>    # Start from last non-leaf, sift down each
>    for i in range((len(arr) - 2) // 2, -1, -1):
>        self._sift_down(i)
>```

>[!example]- JavaScript
>```javascript
>heapify(arr) {
>    this.heap = [...arr];
>    // Start from last non-leaf, sift down each
>    for (let i = Math.floor((this.heap.length - 2) / 2); i >= 0; i--) {
>        this.siftDown(i);
>    }
>}
>```

**Why O(n) not O(n log n)**:
- Half the nodes are leaves (no sift down needed)
- Quarter are one level up (sift down at most 1)
- Eighth are two levels up (sift down at most 2)
- Sum: n/2·0 + n/4·1 + n/8·2 + ... = O(n)

## Python's heapq Module

```python
import heapq

# Create heap from list
nums = [3, 1, 4, 1, 5, 9]
heapq.heapify(nums)  # O(n), in-place

# Operations
heapq.heappush(nums, 2)     # Push
smallest = heapq.heappop(nums)  # Pop min
smallest = nums[0]           # Peek (just index)

# K largest/smallest
heapq.nlargest(k, nums)     # O(n + k log n)
heapq.nsmallest(k, nums)    # O(n + k log n)

# Push and pop in one operation
heapq.heappushpop(nums, val)  # Push, then pop (more efficient)
heapq.heapreplace(nums, val)  # Pop, then push
```

### Max-Heap with heapq

heapq only provides min-heap. For max-heap, negate values:

```python
import heapq

# Max-heap simulation
max_heap = []
heapq.heappush(max_heap, -val)  # Insert negated
max_val = -heapq.heappop(max_heap)  # Negate on extraction
```

## Complexity Summary

| Operation | Time | Notes |
|-----------|------|-------|
| Push | O(log n) | Sift up at most height |
| Pop | O(log n) | Sift down at most height |
| Peek | O(1) | Root access |
| Heapify | O(n) | Not O(n log n) |
| Find arbitrary | O(n) | No ordering for search |

## Common Pitfalls

1. **Modifying elements in heap**: Breaks heap property—remove and re-insert instead
2. **Forgetting heapq is min-heap only**: Negate for max-heap
3. **Using heappush on non-heap list**: Call heapify first
4. **Assuming sorted order**: Heap is partially ordered, not fully sorted

## Key Interview Patterns

| Pattern | Operation | Example |
|---------|-----------|---------|
| Stream minimum | Push + peek | Running median |
| K largest | Min-heap size k | Top K frequent |
| K smallest | Max-heap size k | K closest points |
| Merge sorted | Push/pop from k lists | Merge K sorted lists |
