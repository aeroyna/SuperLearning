# Callable and Future

How do you get a result back from a thread? `Runnable` return `void`.

## 1. `Callable<V>`
An interface similar to `Runnable`, but:
*   Returns a result of type `V`.
*   Can throw a Checked Exception.

```java
Callable<Integer> task = () -> {
    Thread.sleep(1000);
    return 42;
};
```

## 2. `Future<V>`
When you submit a `Callable` to an executor, you immediately get back a `Future` object.
*   **Concept:** "I don't have the result yet, but here is a handle (a receipt). Come back later to claim it."

### Key Methods
*   `get()`: **Blocks** the calling thread until the result is ready.
*   `get(time, unit)`: Blocks for a specific time, then throws `TimeoutException`.
*   `isDone()`: Checks if complete without blocking.
*   `cancel(boolean mayInterrupt)`: Attempts to cancel execution.

## 3. Exception Handling
If the task throws an Exception, the thread catches it and stores it in the `Future`. When you call `get()`, that exception is re-thrown wrapped in an `ExecutionException`.

```java
Future<Integer> future = executor.submit(task);

try {
    Integer result = future.get(); // Blocks
} catch (ExecutionException e) {
    Throwable rootCause = e.getCause(); // The actual exception thrown by task
}
```