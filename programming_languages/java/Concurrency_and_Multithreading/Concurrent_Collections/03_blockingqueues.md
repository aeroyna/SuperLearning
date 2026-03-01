# BlockingQueues

`BlockingQueue` is the cornerstone of the **Producer-Consumer** pattern in Java. It handles all the complex `wait()` and `notify()` logic internally, allowing developers to focus on business logic.

## 1. The Contract
A Queue that supports operations that wait for the queue to become non-empty when retrieving an element, and wait for space to become available when storing an element.

| Operation | Throws Exception | Special Value | Blocks (Waits) | Times Out |
| :--- | :--- | :--- | :--- | :--- |
| **Insert** | `add(e)` | `offer(e)` | `put(e)` | `offer(e, time, unit)` |
| **Remove** | `remove()` | `poll()` | `take()` | `poll(time, unit)` |
| **Examine** | `element()` | `peek()` | N/A | N/A |

*   **`put()`:** If full, the calling thread sleeps until space opens up.
*   **`take()`:** If empty, the calling thread sleeps until an item arrives.

## 2. Implementations

### `ArrayBlockingQueue`
*   **Structure:** Backed by a fixed-size array.
*   **Fairness:** Can optionally enforce FIFO ordering for waiting threads (fairness), preventing starvation but reducing throughput.
*   **Use:** When you need a bounded buffer to prevent memory exhaustion (backpressure).

### `LinkedBlockingQueue`
*   **Structure:** Linked nodes.
*   **Capacity:** Optionally bounded (default is `Integer.MAX_VALUE` - essentially unbounded).
*   **Locking:** Uses separate locks for `put` and `take`, allowing higher concurrency than `ArrayBlockingQueue`.

### `PriorityBlockingQueue`
*   **Structure:** Priority Heap.
*   **Logic:** `take()` always returns the highest priority element. Unbounded.

### `SynchronousQueue`
*   **Structure:** Capacity of **ZERO**.
*   **Logic:** A `put` must wait for a `take`, and vice versa. It is a direct hand-off mechanism.
*   **Use:** `Executors.newCachedThreadPool()` uses this. If a thread is free, hand off the task. If not, create a new thread.

## 3. Producer-Consumer Pattern
```java
BlockingQueue<String> queue = new ArrayBlockingQueue<>(10);

// Producer
new Thread(() -> {
    while(true) queue.put(produceData()); // Blocks if full
}).start();

// Consumer
new Thread(() -> {
    while(true) process(queue.take()); // Blocks if empty
}).start();
```
*   Zero synchronization code written by the developer. Safe and efficient.