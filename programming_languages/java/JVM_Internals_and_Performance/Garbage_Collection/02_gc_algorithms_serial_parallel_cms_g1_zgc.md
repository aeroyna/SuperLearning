# GC Algorithms: Evolution of Collectors

Java provides several GC algorithms, each optimized for different use cases (Throughput vs. Latency).

## 1. Serial GC (`-XX:+UseSerialGC`)
*   **Mechanism:** Uses a single thread for GC. Pauses everything.
*   **Use Case:** Small CLI apps, environments with single CPU core.

## 2. Parallel GC (`-XX:+UseParallelGC`)
*   **Mechanism:** Multiple threads for Minor GC. Multiple threads for Major GC (Java 8+ default).
*   **Goal:** **Throughput**. Maximize total work done over time. Ideally suited for batch processing / analytics.
*   **Downside:** Long pause times during Full GC.

## 3. CMS (Concurrent Mark Sweep) - *Deprecated*
*   **Goal:** **Low Latency**.
*   **Mechanism:** Performs most of the "Marking" work *concurrently* while the application is running. Does not compact memory (fragmentation issues).
*   **Status:** Removed in Java 14.

## 4. G1 GC (Garbage First) (`-XX:+UseG1GC`)
*   **Default:** Since Java 9.
*   **Structure:** Doesn't divide Heap into fixed Young/Old blocks. Instead, divides Heap into thousands of small **Regions**.
*   **Logic:** It identifies regions with the *most garbage* ("Garbage First") and cleans them. It can compact memory on the fly.
*   **Goal:** Balance between Throughput and Latency. Predictable pause times (e.g., "Pause no more than 200ms").

## 5. ZGC (Z Garbage Collector) (`-XX:+UseZGC`)
*   **Status:** Production ready in Java 15+.
*   **Goal:** **Ultra-Low Latency**. Sub-millisecond pauses, regardless of Heap size (even Terabytes).
*   **Mechanism:** Uses "Colored Pointers" and Load Barriers to handle object relocation concurrently.
*   **Use Case:** High-frequency trading, massive heaps, real-time APIs.

## 6. Shenandoah
*   Similar goals to ZGC (sub-millisecond pauses). Developed by RedHat.