# Wait, Notify, and NotifyAll

This is the low-level mechanism for **Inter-Thread Communication**. While effective, it is complex and brittle. Modern Java prefers `java.util.concurrent` (locks/conditions), but understanding this explains how everything else is built.

## 1. The Concept: The Wait Set
Every Object Monitor has a "Wait Set".
*   **`wait()`:** "I cannot proceed (e.g., queue is full). I will release the lock, go to sleep, and join the Wait Set."
*   **`notify()`:** "Something changed (e.g., I removed an item). Wake up *one* random thread in the Wait Set."
*   **`notifyAll()`:** "Something changed. Wake up *everyone* in the Wait Set."

## 2. The Rules (Strict)
1.  **Must hold lock:** You can only call `wait()`/`notify()` inside a `synchronized` block on that specific object. If not -> `IllegalMonitorStateException`.
2.  **Releases Lock:** Calling `wait()` atomically releases the lock so others can enter.
3.  **Re-acquires Lock:** When a thread wakes up (after notify), it must re-acquire the lock before returning from the `wait()` method.

## 3. The Pattern (Standard Idiom)

### The Producer (Notify)
```java
synchronized(lock) {
    // 1. Change state
    dataReady = true;
    // 2. Notify waiting threads
    lock.notifyAll();
}
```

### The Consumer (Wait)
```java
synchronized(lock) {
    // 1. Check condition in a WHILE LOOP (Not IF)
    while (!dataReady) {
        lock.wait(); 
    }
    // 2. Proceed
    processData();
}
```

### Why `while` loop? (Spurious Wakeups)
The OS can sometimes wake up a thread for no reason ("Spurious Wakeup"). Or, `notifyAll` woke up 10 threads, but only 1 could consume the data.
*   If you use `if`, the thread wakes up, assumes data is ready, and crashes.
*   If you use `while`, the thread wakes up, *re-checks* the condition, sees data is missing, and goes back to sleep.

## 4. `notify()` vs `notifyAll()`
*   **`notify()`:** Efficient (wakes 1 thread). Dangerous: if you wake the wrong thread (e.g., a producer wakes another producer when it should wake a consumer), the system can stall (Lost Wakeup).
*   **`notifyAll()`:** Safe. Wakes everyone. Everyone checks their condition. The winner proceeds, losers go back to sleep.
*   **Best Practice:** Always use `notifyAll()` unless you have a specific, performance-critical, homogeneous thread scenario.