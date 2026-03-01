# Threads

## Overview

Threads are lightweight processes that share the same address space. Understanding threads is crucial for modern concurrent programming and system design.

## Topics Covered

1. **[Thread Concept](01_thread_concept.md)**
   - What is a thread?
   - Process vs Thread
   - Benefits of threads (responsiveness, resource sharing, economy, scalability)
   - Thread components
   - Shared vs private resources

2. **[Multithreading Models](02_multithreading_models.md)**
   - User threads vs Kernel threads
   - Many-to-One model
   - One-to-One model
   - Many-to-Many model
   - Two-level model
   - Trade-offs of each model

3. **[Thread Libraries](03_thread_libraries.md)**
   - POSIX Pthreads
   - Java threads
   - Windows threads
   - Thread creation, joining, and termination
   - Code examples

4. **[Threading Issues](04_threading_issues.md)**
   - fork() and exec() semantics with threads
   - Thread cancellation (asynchronous vs deferred)
   - Signal handling in multithreaded programs
   - Thread pools
   - Thread-local storage

## Key Takeaways

- Threads share address space but have separate stacks
- Context switching between threads is faster than between processes
- Multithreading models determine the relationship between user and kernel threads
- Thread synchronization is crucial to avoid race conditions

## Interview Focus

- Compare processes and threads (when to use each)
- Explain benefits of multithreading
- Understand different multithreading models
- Discuss threading challenges (synchronization, deadlocks)
- Thread pool advantages
