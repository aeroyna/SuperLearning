# Parallel Streams: Fork/Join Under the Hood

The promise of Parallel Streams is "multi-threading for free." The reality is more nuanced.

## 1. The Common Fork/Join Pool
All parallel streams in a JVM application share the **same** global `ForkJoinPool.commonPool()`.
*   **Default Size:** Number of CPU cores - 1.
*   **Risk:** If you have a parallel stream performing a **blocking operation** (e.g., File IO, DB call), you will block threads in the global pool. This can starve *all other parallel streams* in your application.
*   **Rule:** NEVER use parallel streams for blocking tasks. Only for CPU-intensive calculations.

## 2. Spliterators
How does a stream split itself?
*   `ArrayList`: Splits easily (index 0-500, index 501-1000). Perfect parallelism.
*   `LinkedList`: Cannot be split easily. Must traverse to find midpoint. Terrible parallelism.
*   **Conclusion:** Data structure matters. Array-based sources parallelize best.

## 3. Overhead vs. Benefit
Parallelism has overhead:
1.  Splitting the task.
2.  Dispatching to threads.
3.  Merging results (especially if using `sorted()` or `collect()` with ordering requirements).

**The "NQ" Model:**
Parallelism pays off when `N * Q > 10,000`.
*   **N:** Number of elements.
*   **Q:** Cost (CPU ops) per element.
*   *Small list, simple op:* Sequential is faster.
*   *Huge list, complex op:* Parallel is faster.
