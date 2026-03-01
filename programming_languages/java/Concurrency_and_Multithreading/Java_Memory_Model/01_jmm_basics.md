# JMM Basics: Working Memory vs Main Memory

## 1. The Abstract Architecture
The JMM defines an abstract computer architecture.

*   **Main Memory:** Shared by all threads. Holds variables (heap objects, statics).
*   **Working Memory:** Private to each thread. Contains a *copy* of variables from Main Memory. Conceptually maps to CPU Registers + L1/L2 Caches.

## 2. Interaction
1.  **Read:** Thread reads `x` from Main Memory to Working Memory.
2.  **Use/Assign:** Thread operates on `x` in Working Memory.
3.  **Write:** Thread flushes `x` from Working Memory back to Main Memory.

**The Problem:** The JVM does not guarantee *when* the write happens. It could be nanoseconds, or seconds. Until it happens, other threads see stale data (Visibility Problem).