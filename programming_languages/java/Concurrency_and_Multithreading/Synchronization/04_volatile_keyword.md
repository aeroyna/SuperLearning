# The Volatile Keyword

`volatile` is widely misunderstood. It is a lightweight synchronization mechanism that provides **Memory Visibility** but *not* Atomicity.

## 1. The Problem: Memory Visibility
In modern architecture, each CPU core has its own Cache (L1, L2, L3).
*   **Thread A (Core 1):** Reads variable `flag` from RAM into Cache. Changes it to `true` in Cache.
*   **Thread B (Core 2):** Reads variable `flag` from RAM. It still sees `false` because Thread A hasn't flushed its cache to RAM yet.

This is a "Visibility" problem. The value has changed, but other threads can't see it.

## 2. The Solution: `volatile`
Declaring a variable `volatile` tells the JVM and CPU:
> *"Do not cache this variable in CPU registers or local caches. Always read from and write directly to Main Memory."*

```java
private volatile boolean running = true;

public void run() {
    while (running) { // Guaranteed to see the latest value
        // work
    }
}

public void stop() {
    running = false; // Write is immediately visible to other threads
}
```

## 3. What `volatile` Does NOT Do (Atomicity)
`volatile` guarantees that *reads* and *writes* are visible. It does **not** make compound actions atomic.

```java
volatile int count = 0;

// Thread 1 & 2
count++; 
```
*   `count++` is still Read-Modify-Write.
*   Thread 1 reads 0 from RAM.
*   Thread 2 reads 0 from RAM.
*   Thread 1 writes 1 to RAM.
*   Thread 2 writes 1 to RAM.
*   **Result:** Data loss. `volatile` does not fix this. Use `AtomicInteger` or `synchronized` for counters.

## 4. The "Happens-Before" Guarantee
`volatile` establishes a "Happens-Before" relationship.
*   **Rule:** A write to a `volatile` field *happens-before* every subsequent read of that same field.
*   **Piggybacking:** Any variable written *before* writing to the volatile variable will also be flushed to main memory. This allows `volatile` to act as a memory barrier.

## 5. When to use?
1.  **Flags:** Status flags (boolean `running`, `shutdown`).
2.  **One Writer, Many Readers:** If only one thread updates the value, but many read it, `volatile` is safe and faster than locks.
3.  **Double-Checked Locking:** Used in Singleton patterns to prevent partially constructed objects.