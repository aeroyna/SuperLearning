# Concurrency and Multithreading

Modern computing relies heavily on parallelism. C++ provides a robust standard library for writing multithreaded applications. This chapter covers the creation and management of threads, synchronization mechanisms to prevent race conditions (mutexes, locks, condition variables), and high-level concurrency tools like futures and promises.

You will learn about:
- **Thread Management:** Creating, joining, and detaching threads.
- **Synchronization:** Protecting shared data with mutexes and locks.
- **Inter-thread Communication:** Using condition variables.
- **Atomics:** Lock-free programming for simple types.
- **Task-based Concurrency:** Using `std::async` and `std::future`.
- **Thread Pools:** Building a thread pool from standard primitives.

## In this chapter

- **[Threads](01_threads.md)**
- **[Mutexes](02_mutexes.md)**
- **[Condition Variables](03_condition_variables.md)**
- **[Atomics](04_atomics.md)**
- **[Async and Futures](05_async_and_futures.md)**
- **[Thread Pools](06_thread_pools.md)**
- **[Practice Problems](practice_problems.md)**
