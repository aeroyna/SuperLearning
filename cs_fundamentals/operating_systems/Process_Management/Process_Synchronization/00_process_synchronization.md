# Process Synchronization

## Overview

When multiple processes/threads access shared resources concurrently, synchronization is needed to ensure correctness. This is one of the most important interview topics.

## Topics Covered

1. **[Critical Section Problem](01_critical_section_problem.md)**
   - Race condition
   - Critical section definition
   - Requirements: Mutual exclusion, progress, bounded waiting
   - Preemptive vs non-preemptive kernels

2. **[Peterson's Solution](02_petersons_solution.md)**
   - Software solution for two processes
   - Turn and flag variables
   - Proof of correctness
   - Limitations

3. **[Synchronization Hardware](03_synchronization_hardware.md)**
   - Hardware support for synchronization
   - Test-and-Set instruction
   - Compare-and-Swap instruction
   - Atomic operations
   - Memory barriers

4. **[Mutex Locks](04_mutex_locks.md)**
   - Mutual exclusion lock
   - acquire() and release()
   - Spinlocks vs blocking locks
   - Busy waiting
   - Implementation

5. **[Semaphores](05_semaphores.md)**
   - Counting semaphore
   - Binary semaphore (vs mutex)
   - wait() and signal() operations
   - Implementation with and without busy waiting
   - Use cases

6. **[Monitors](06_monitors.md)**
   - High-level synchronization construct
   - Condition variables
   - wait() and signal() in monitors
   - Monitor vs semaphore
   - Java synchronized keyword

7. **[Classic Synchronization Problems](07_classic_problems.md)**
   - Producer-Consumer (Bounded Buffer) Problem
   - Readers-Writers Problem
   - Dining Philosophers Problem
   - Solutions using semaphores and monitors

## Key Takeaways

- Race conditions occur when outcome depends on execution timing
- Critical sections must have mutual exclusion
- Semaphores are more general than mutexes
- Monitors provide structured synchronization
- Classic problems demonstrate common synchronization patterns

## Interview Focus

- Identify and solve race conditions
- Implement critical sections using different primitives
- Compare mutex, semaphore, and monitor
- Solve classic synchronization problems
- Understand deadlock potential in synchronization
