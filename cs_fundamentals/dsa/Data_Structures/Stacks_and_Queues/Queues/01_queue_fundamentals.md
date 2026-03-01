## Queue Fundamentals (FIFO)

A queue is a linear data structure that follows the **FIFO (First-In, First-Out)** principle. The first element added to the queue will be the first element to be removed.

### Core Idea & Analogy
Think of a checkout line at a grocery store or a queue for a ride at an amusement park. The first person to get in line is the first person to be served. New people always join at the back of the line and wait their turn.

### Key Operations
A queue supports three main O(1) operations:
- **Enqueue**: Add an element to the back (tail) of the queue.
- **Dequeue**: Remove and return the element from the front (head) of the queue.
- **Peek**: Look at the front element without removing it.

### Implementation

>[!example]- C++
>```cpp
>#include <queue>
>
>// Initialize an empty queue
>std::queue<int> queue;
>
>// Enqueue operations
>queue.push(10);  // queue has 10
>queue.push(20);  // queue has 10, 20
>queue.push(30);  // queue has 10, 20, 30
>
>// Peek operation
>if (!queue.empty()) {
>    int front_element = queue.front();
>    // front_element is 10
>}
>
>// Dequeue operation
>if (!queue.empty()) {
>    int removed_element = queue.front();
>    queue.pop();
>    // removed_element is 10, queue now has 20, 30
>}
>
>// Check if empty
>bool is_empty = queue.empty(); // false
>```

>[!example]- Java
>```java
>import java.util.LinkedList;
>import java.util.Queue;
>
>// Initialize an empty queue
>Queue<Integer> queue = new LinkedList<>();
>
>// Enqueue operations
>queue.offer(10);  // queue is [10]
>queue.offer(20);  // queue is [10, 20]
>queue.offer(30);  // queue is [10, 20, 30]
>
>// Peek operation
>if (!queue.isEmpty()) {
>    int frontElement = queue.peek();
>    // frontElement is 10
>}
>
>// Dequeue operation
>if (!queue.isEmpty()) {
>    int removedElement = queue.poll();
>    // removedElement is 10, queue is now [20, 30]
>}
>
>// Check if empty
>boolean isEmpty = queue.isEmpty(); // false
>```

>[!example]- Python
>```python
>from collections import deque
>
># Initialize an empty queue
>queue = deque()
>
># Enqueue operations
>queue.append(10)  # queue is deque([10])
>queue.append(20)  # queue is deque([10, 20])
>queue.append(30)  # queue is deque([10, 20, 30])
>
># Peek operation
>if queue:
>    front_element = queue[0]
>    # front_element is 10
>
># Dequeue operation
>if queue:
>    removed_element = queue.popleft()
>    # removed_element is 10, queue is now deque([20, 30])
>
># Check if empty
>is_empty = not queue # False
>```

>[!example]- JavaScript
>```javascript
>// Initialize an empty queue (using array)
>const queue = [];
>
>// Enqueue operations
>queue.push(10);  // queue is [10]
>queue.push(20);  // queue is [10, 20]
>queue.push(30);  // queue is [10, 20, 30]
>
>// Peek operation
>if (queue.length > 0) {
>    const frontElement = queue[0];
>    // frontElement is 10
>}
>
>// Dequeue operation
>if (queue.length > 0) {
>    const removedElement = queue.shift();
>    // removedElement is 10, queue is now [20, 30]
>}
>
>// Check if empty
>const isEmpty = queue.length === 0; // false
>```

### Implementation Notes by Language

**Python**: Using a standard Python `list` is **inefficient** for a queue because removing from the beginning (`list.pop(0)`) takes O(n) time. The correct, high-performance way to implement a queue is with the `collections.deque` object (pronounced "deck," short for double-ended queue).
- A `deque` is optimized for fast appends and pops from both ends, making it perfect for a queue implementation.
- `deque.append()` is the **enqueue** operation (add to the right/back).
- `deque.popleft()` is the **dequeue** operation (remove from the left/front).
- Both operations have an O(1) time complexity.

**C++**: Use `std::queue` from the `<queue>` header.
- `queue.push()` enqueues to the back.
- `queue.pop()` dequeues from the front (returns void).
- `queue.front()` peeks at the front element.
- `queue.back()` peeks at the back element.

**Java**: Use `Queue<T>` interface with `LinkedList` implementation.
- `queue.offer()` or `queue.add()` enqueues to the back.
- `queue.poll()` dequeues from the front (returns null if empty).
- `queue.peek()` looks at the front without removing.
- `queue.isEmpty()` checks if empty.

**JavaScript**: Arrays can be used but `shift()` is O(n).
- `array.push()` adds to the back.
- `array.shift()` removes from the front (O(n) operation).
- `array[0]` peeks at the front.
- For better performance, consider using a circular buffer or linked list implementation.

### When to Use a Queue
The primary and most critical use case for a queue in interviews is to implement **Breadth-First Search (BFS)** for trees and graphs. The FIFO property perfectly manages the nodes at each "level" of the traversal, ensuring that you explore the graph layer by layer.
