# Queues

A queue is a First-In-First-Out (FIFO) data structure that models waiting lines—elements are processed in the order they arrive. This property makes queues essential for breadth-first algorithms and scheduling systems.

## Overview

Queues support three primary operations:
- **Enqueue**: Add element to back - O(1)
- **Dequeue**: Remove element from front - O(1)
- **Peek**: View front element without removal - O(1)

## Topics

- [5.3.1 Queue Fundamentals](01_queue_fundamentals.md)

## Internal Implementation

### Array-Based Circular Queue

A naive array implementation has O(n) dequeue (shifting elements). Circular queues solve this:

>[!example]- C++
>```cpp
>class CircularQueue {
>    vector<int> data;
>    int front = 0, rear = 0, size = 0, capacity;
>public:
>    CircularQueue(int k) : capacity(k) {
>        data.resize(k);
>    }
>    bool enqueue(int val) {
>        if (isFull()) return false;
>        data[rear] = val;
>        rear = (rear + 1) % capacity;
>        size++;
>        return true;
>    }
>    bool dequeue() {
>        if (isEmpty()) return false;
>        front = (front + 1) % capacity;
>        size--;
>        return true;
>    }
>    int getFront() {
>        return isEmpty() ? -1 : data[front];
>    }
>    bool isFull() { return size == capacity; }
>    bool isEmpty() { return size == 0; }
>};
>```

>[!example]- Java
>```java
>class CircularQueue {
>    private int[] data;
>    private int front = 0, rear = 0, size = 0;
>    
>    public CircularQueue(int k) {
>        data = new int[k];
>    }
>    public boolean enqueue(int val) {
>        if (isFull()) return false;
>        data[rear] = val;
>        rear = (rear + 1) % data.length;
>        size++;
>        return true;
>    }
>    public boolean dequeue() {
>        if (isEmpty()) return false;
>        front = (front + 1) % data.length;
>        size--;
>        return true;
>    }
>    public int Front() {
>        return isEmpty() ? -1 : data[front];
>    }
>    public boolean isFull() { return size == data.length; }
>    public boolean isEmpty() { return size == 0; }
>}
>```

>[!example]- Python
>```python
>class CircularQueue:
>    def __init__(self, capacity):
>        self._data = [None] * capacity
>        self._front = 0
>        self._rear = 0
>        self._size = 0
>
>    def enqueue(self, val):
>        if self._size == len(self._data):
>            raise OverflowError("Queue full")
>        self._data[self._rear] = val
>        self._rear = (self._rear + 1) % len(self._data)
>        self._size += 1
>
>    def dequeue(self):
>        if self._size == 0:
>            raise IndexError("Queue empty")
>        val = self._data[self._front]
>        self._front = (self._front + 1) % len(self._data)
>        self._size -= 1
>        return val
>```

>[!example]- JavaScript
>```javascript
>class CircularQueue {
>    constructor(k) {
>        this.data = new Array(k);
>        this.capacity = k;
>        this.front = 0;
>        this.rear = 0;
>        this.size = 0;
>    }
>    enqueue(val) {
>        if (this.isFull()) return false;
>        this.data[this.rear] = val;
>        this.rear = (this.rear + 1) % this.capacity;
>        this.size++;
>        return true;
>    }
>    dequeue() {
>        if (this.isEmpty()) return false;
>        this.front = (this.front + 1) % this.capacity;
>        this.size--;
>        return true;
>    }
>    Front() {
>        return this.isEmpty() ? -1 : this.data[this.front];
>    }
>    isFull() { return this.size === this.capacity; }
>    isEmpty() { return this.size === 0; }
>}
>```

**Memory layout visualization**:
```
Initial:     [None | None | None | None | None]
              ^front
              ^rear

Enqueue A,B: [A    | B    | None | None | None]
              ^front       ^rear

Dequeue:     [None | B    | None | None | None]
                    ^front ^rear

Enqueue C,D,E: [None | B | C | D | E]
                      ^front        ^rear

Wrap around:
Enqueue F:   [F    | B | C | D | E]
              ^rear ^front
```

**Why circular**: Without wrap-around, front would advance indefinitely, wasting memory.

### Deque (Double-Ended Queue)

```python
from collections import deque

dq = deque()
dq.append(x)      # Add to right - O(1)
dq.appendleft(x)  # Add to left - O(1)
dq.pop()          # Remove from right - O(1)
dq.popleft()      # Remove from left - O(1)
```

**Internal structure**: Python's deque uses a doubly-linked list of fixed-size blocks (typically 64 elements each). This provides:
- O(1) operations at both ends
- Better cache locality than pure linked list
- No resizing/copying like arrays

## Queue Variants

### Priority Queue

Elements dequeued by priority, not arrival order. See [Heaps](../../Heaps_and_Priority_Queues/00_heaps_and_priority_queues.md).

### Monotonic Queue

Maintains elements in sorted order within a sliding window. See [Monotonic Stack](../Monotonic_Stack/00_monotonic_stack.md).

## Common Queue Patterns

### Pattern 1: BFS Traversal

```python
def bfs(root):
    if not root:
        return []

    result = []
    queue = deque([root])

    while queue:
        node = queue.popleft()
        result.append(node.val)

        if node.left:
            queue.append(node.left)
        if node.right:
            queue.append(node.right)

    return result
```

### Pattern 2: Level-Order Processing

```python
def level_order(root):
    if not root:
        return []

    result = []
    queue = deque([root])

    while queue:
        level_size = len(queue)
        level = []
        for _ in range(level_size):
            node = queue.popleft()
            level.append(node.val)
            if node.left:
                queue.append(node.left)
            if node.right:
                queue.append(node.right)
        result.append(level)

    return result
```

**Key insight**: Capturing `level_size` before processing ensures we only process nodes from the current level in each iteration.

### Pattern 3: Multi-Source BFS

```python
def walls_and_gates(rooms):
    """Fill each empty room with distance to nearest gate."""
    if not rooms:
        return

    m, n = len(rooms), len(rooms[0])
    INF = 2147483647
    queue = deque()

    # Start BFS from all gates simultaneously
    for i in range(m):
        for j in range(n):
            if rooms[i][j] == 0:  # Gate
                queue.append((i, j))

    while queue:
        r, c = queue.popleft()
        for dr, dc in [(0, 1), (0, -1), (1, 0), (-1, 0)]:
            nr, nc = r + dr, c + dc
            if 0 <= nr < m and 0 <= nc < n and rooms[nr][nc] == INF:
                rooms[nr][nc] = rooms[r][c] + 1
                queue.append((nr, nc))
```

## Implementation Trade-offs

| Implementation | Enqueue | Dequeue | Space | Use Case |
|---------------|---------|---------|-------|----------|
| List (naive) | O(1) | O(n) | O(n) | Never use |
| Circular array | O(1)* | O(1) | O(n) | Fixed size |
| Linked list | O(1) | O(1) | O(n) + pointers | Unbounded |
| collections.deque | O(1) | O(1) | O(n) | Default choice |

*Amortized if dynamic resizing

## Common Pitfalls

1. **Using list.pop(0)**: This is O(n), use `collections.deque.popleft()`
2. **Forgetting visited set**: In BFS, always track visited to avoid cycles
3. **Processing levels incorrectly**: Capture level size before iteration
4. **Empty queue check**: Always verify queue is non-empty before dequeue

## Key Interview Problems

| Problem | Pattern | Difficulty | LeetCode Link |
| --------- | --------- | ------------ | --- |
| Binary Tree Level Order | Level-order | Medium | [Link](https://leetcode.com/problems/binary-tree-level-order/) |
| Number of Islands (BFS) | Grid BFS | Medium | [Link](https://leetcode.com/problems/number-of-islands-bfs/) |
| Rotting Oranges | Multi-source BFS | Medium | [Link](https://leetcode.com/problems/rotting-oranges/) |
| Word Ladder | BFS + string | Hard | [Link](https://leetcode.com/problems/word-ladder/) |
| Sliding Window Maximum | Monotonic deque | Hard | [Link](https://leetcode.com/problems/sliding-window-maximum/) |
