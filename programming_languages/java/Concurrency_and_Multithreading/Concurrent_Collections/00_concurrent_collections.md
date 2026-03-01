# Concurrent Collections

The `java.util.concurrent` package (introduced in Java 5) provided highly optimized, thread-safe collections. They solve the performance bottlenecks of the old legacy collections (`Vector`, `Hashtable`) and the fragility of `Collections.synchronizedList()`.

## In this chapter, you will learn:
*   [**ConcurrentHashMap**](01_concurrenthashmap.md): Lock stripping (Java 7) vs CAS/synchronized buckets (Java 8+).
*   [**CopyOnWriteArrayList**](02_copyonwritearraylist.md): Snapshot iterators for read-heavy workloads.
*   [**BlockingQueues**](03_blockingqueues.md): Implementing Producer-Consumer patterns.
