# Queue and Deque

The `Queue` interface is designed for holding elements prior to processing. `Deque` (Double Ended Queue) extends `Queue` to support insertion and removal at both ends.

## 1. `Queue` Interface

Typically orders elements in a **FIFO (First-In-First-Out)** manner (except `PriorityQueue`).

### Key Methods (Two Forms)
Queue methods come in two forms: one throws an exception if the operation fails, the other returns a special value (`null` or `false`).

| Operation | Throws Exception | Returns Special Value |
| :--- | :--- | :--- |
| **Insert** | `add(e)` | `offer(e)` |
| **Remove** | `remove()` | `poll()` |
| **Examine** | `element()` | `peek()` |

### `PriorityQueue`
*   **Ordering:** Elements are ordered by their **priority** (natural order or Comparator). The head is always the "least" element.
*   **Structure:** Priority Heap.
*   **Use Case:** Task scheduling, Dijkstra's algorithm.

```java
Queue<Integer> pq = new PriorityQueue<>();
pq.add(10);
pq.add(5);
pq.add(20);

System.out.println(pq.poll()); // 5 (Smallest first)
System.out.println(pq.poll()); // 10
```

## 2. `Deque` Interface

"Deque" (pronounced "deck") stands for **Double Ended Queue**. It can function as a Queue (FIFO) or a Stack (LIFO).

### Implementations
1.  **`ArrayDeque`:**
    *   Resizable array implementation.
    *   **Faster than `Stack` and `LinkedList`** for stack/queue operations.
    *   No null elements.
2.  **`LinkedList`:**
    *   Implements `Deque`. Allows nulls.

### Using Deque as a Stack (LIFO)
The legacy `Stack` class is synchronized and slow. **Use `ArrayDeque` instead.**

```java
Deque<String> stack = new ArrayDeque<>();
stack.push("Bottom");
stack.push("Top");

System.out.println(stack.pop()); // "Top"
```

## 3. BlockingQueues (Concurrency)
Interfaces like `BlockingQueue` (in `java.util.concurrent`) are thread-safe queues that wait for the queue to become non-empty when retrieving an element, and wait for space to become available when storing an element. Crucial for Producer-Consumer problems.
