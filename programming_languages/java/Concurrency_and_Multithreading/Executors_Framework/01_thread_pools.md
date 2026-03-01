# Thread Pools

A Thread Pool acts as a reservoir of worker threads.

## 1. The Mechanic
1.  **Pool Creation:** A pool starts with N threads.
2.  **Task Submission:** You submit a `Runnable`.
3.  **Queueing:** If a thread is available, it takes the task. If not, the task is placed in a `BlockingQueue`.
4.  **Reuse:** When a thread finishes a task, it doesn't die. It goes back to the queue to check for more work.

## 2. Common Pool Types (`Executors` Factory)

### `newFixedThreadPool(n)`
*   **Structure:** N threads. Unbounded Queue (`LinkedBlockingQueue`).
*   **Behavior:** Creates threads up to N. If all busy, tasks queue up forever.
*   **Use Case:** Predictable load, servers with resource limits.

### `newCachedThreadPool()`
*   **Structure:** 0 to Integer.MAX_VALUE threads. `SynchronousQueue`.
*   **Behavior:** If a thread is free, reuse it. If not, create a new one. Threads idle for 60s are killed.
*   **Use Case:** Many short-lived asynchronous tasks. **Dangerous** if tasks are long-running (can crash heap).

### `newSingleThreadExecutor()`
*   **Structure:** 1 Thread. Unbounded Queue.
*   **Behavior:** Ensures tasks are executed sequentially. Safe alternative to a manual background thread.

## 3. Custom `ThreadPoolExecutor`
For production, avoid the factory methods (which use unbounded queues) and build your own.

```java
new ThreadPoolExecutor(
    10, // Core Pool Size (min threads)
    20, // Max Pool Size (max threads)
    60L, TimeUnit.SECONDS, // Keep-alive time for excess threads
    new ArrayBlockingQueue<>(100), // Bounded Queue (Critical for stability)
    new ThreadPoolExecutor.CallerRunsPolicy() // Rejection Policy
);
```

### Rejection Policies
What happens if the Queue is full AND Max Threads are busy?
*   `AbortPolicy`: Throw `RejectedExecutionException`.
*   `CallerRunsPolicy`: The thread submitting the task runs it. This provides simple **backpressure** (slows down the producer).
*   `DiscardPolicy`: Silently drop the task.