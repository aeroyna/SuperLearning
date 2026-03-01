# Deadlocks and Livelocks

Deadlock is the ultimate nightmare of concurrency: the application is technically running (threads exist), but it has ceased to function.

## 1. Deadlock Anatomy
A deadlock occurs when two (or more) threads are waiting for each other to release a resource, forming a cycle.

**The Coffman Conditions (All 4 needed for deadlock):**
1.  **Mutual Exclusion:** Resources can be held by only one thread.
2.  **Hold and Wait:** A thread holding a resource waits for another.
3.  **No Preemption:** Resources cannot be forcibly taken away.
4.  **Circular Wait:** A waits for B, B waits for A.

### Classic Example
*   **Thread 1:** Holds Lock A, wants Lock B.
*   **Thread 2:** Holds Lock B, wants Lock A.

```java
// Thread 1
synchronized(A) {
    synchronized(B) { ... } // Blocks waiting for B
}

// Thread 2
synchronized(B) {
    synchronized(A) { ... } // Blocks waiting for A
}
```

## 2. Prevention Strategies

### A. Lock Ordering (The Silver Bullet)
Always acquire locks in a consistent global order.
*   If *every* thread requires Lock A before Lock B, deadlock is impossible because the "Circular Wait" condition is broken.
*   *Implementation:* If locking generic objects (like `Account`), order them by ID (e.g., lock `fromAccount` then `toAccount` if `from.id < to.id`).

### B. Timed Locks (`tryLock`)
Using explicit locks (`ReentrantLock`), you can use `.tryLock(timeout)`.
*   If a thread can't get a lock within 2 seconds, it gives up (backs off) and releases its own locks, allowing the other thread to proceed.

## 3. Livelock
A state where threads are not blocked (they are `RUNNABLE`), but they are busy continuously responding to each other without making progress.
*   *Analogy:* Two people meeting in a hallway. Both step left. Both step right. Both step left. Repeatedly.
*   *Fix:* Introduce randomness (Thread.sleep(random)) into the retry logic.

## 4. Starvation
A thread is perpetually denied access to resources because "greedy" threads are constantly grabbing them.
*   *Example:* High-priority threads constantly preempting low-priority threads.
*   *Fix:* Fairness policies (Fair Locks).