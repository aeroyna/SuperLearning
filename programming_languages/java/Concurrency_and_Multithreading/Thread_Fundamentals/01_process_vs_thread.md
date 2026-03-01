# Process vs Thread: Deep Dive

This file explores the architectural differences between Processes and Threads, focusing on memory models and communication costs.

## 1. Architectural Diagram

```text
+---------------- PROCESS A ----------------+   +---- PROCESS B ----+
|  [ Code Segment ] [ Data Segment ]        |   |                   |
|  [ Open Files   ] [ Signal Handlers ]     |   |                   |
|                                           |   |                   |
|  +--- Thread 1 ---+   +--- Thread 2 ---+  |   |                   |
|  | [ Registers ]  |   | [ Registers ]  |  |   |                   |
|  | [ PC        ]  |   | [ PC        ]  |  |   |                   |
|  | [ Stack     ]  |   | [ Stack     ]  |  |   |                   |
|  +----------------+   +----------------+  |   |                   |
|          |                    |           |   |                   |
|          +--- SHARED HEAP ----+           |   |                   |
+-------------------------------------------+   +-------------------+
```

## 2. Memory Anatomy

### The Stack (Thread-Local)
*   **What it holds:** Primitive local variables (`int i = 5`), object references (`String s = ...`), and method call frames.
*   **Isolation:** Thread 1 cannot access Thread 2's stack.
*   **Implication:** You never need to synchronize local variables. They are inherently thread-safe.
*   **Stack Overflow:** If a thread calls methods too deeply (recursion), it blows its *own* stack, throwing `StackOverflowError`. It does not affect other threads.

### The Heap (Process-Global)
*   **What it holds:** All Objects (`new Something()`), class metadata, static variables.
*   **Sharing:** If Thread 1 has a reference to an object `X`, and Thread 2 has a reference to the *same* object `X`, they both access the same memory address in the Heap.
*   **Implication:** This is where **Race Conditions** happen. If both threads try to write to `X.field` at the same time, the data can be corrupted.

## 3. Inter-Thread Communication (Shared Memory)
Because threads share the Heap, communication is implicit and fast.
*   **Thread A:** `sharedObject.value = 10;`
*   **Thread B:** `int x = sharedObject.value;`

**The Hidden Cost: Visibility**
While fast, it's not instant. Modern CPUs have local caches (L1/L2). Thread A might write `10` to its *local cache*, not main RAM. Thread B might read its *own local cache* and see the old value `0`.
*   **Nuance:** We need mechanisms (like `volatile` or `synchronized`) not just for mutual exclusion, but to force **Memory Visibility** (flushing caches to RAM).

## 4. Summary Table

| Feature | Process | Thread |
| :--- | :--- | :--- |
| **Address Space** | Independent | Shared (Heap) |
| **Creation Cost** | High (OS calls, memory map) | Medium (Stack allocation, OS calls) |
| **Communication** | Slow (IPC) | Fast (Shared Memory) |
| **Safety** | High (Crash is isolated) | Low (One thread crash can kill process*) |
| **Context Switch**| Expensive | Moderate |

*\*Note: In Java, an Unchecked Exception kills only the Thread, not the JVM. However, an `OutOfMemoryError` caused by one thread affects the whole Heap, crashing the JVM.*