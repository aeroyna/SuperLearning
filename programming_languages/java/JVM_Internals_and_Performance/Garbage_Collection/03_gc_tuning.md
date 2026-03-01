# GC Tuning: Basics

Tuning GC is a dark art, but basic principles apply.

## 1. Key Metrics
*   **Latency:** How long does the app freeze during GC? (Important for web apps).
*   **Throughput:** What % of CPU time is spent running app code vs GC code? (Important for batch jobs).
*   **Footprint:** How much RAM does it need?

## 2. Basic Flags
*   `-Xms`: Initial Heap Size.
*   `-Xmx`: Max Heap Size. *Best practice: Set Xms and Xmx to the same value to avoid resizing overhead.*
*   `-Xlog:gc*`: Enable GC logging (Java 9+). Mandatory for debugging.

## 3. Tuning Strategy (G1GC)
1.  **Set the Goal:** `-XX:MaxGCPauseMillis=200` (Default).
2.  **Give it Memory:** G1 needs headroom to move objects around. Don't run it tight on memory.
3.  **Avoid Humongous Allocations:** Objects > 50% of a region size are "Humongous" and expensive. Increase region size (`-XX:G1HeapRegionSize`) if logs show many humongous allocations.

## 4. Common Problems
*   **Memory Leak:** Heap fills up with reachable objects. GC runs constantly (High CPU) but reclaims nothing. -> `OutOfMemoryError`.
    *   *Fix:* Heap Dump analysis (Eclipse MAT, VisualVM).
*   **Promotion Failure:** Old Gen fills up faster than GC can clean it.
*   **Metaspace OOM:** Too many classes loaded.