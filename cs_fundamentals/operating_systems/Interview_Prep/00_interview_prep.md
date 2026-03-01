# Operating Systems Interview Preparation

## Overview

This section consolidates the most important OS concepts for technical interviews at FAANG+ companies.

## Topics Covered

1. **[Top 50 OS Interview Questions](01_top_50_questions.md)**
   - Processes and threads
   - Memory management
   - CPU scheduling
   - Synchronization and deadlocks
   - File systems
   - Virtual memory
   - I/O systems
   - With detailed answers and explanations

2. **[Common Mistakes to Avoid](02_common_mistakes.md)**
   - Confusing process and thread
   - Misunderstanding virtual memory
   - Not explaining trade-offs
   - Forgetting edge cases
   - Poor communication

3. **[FAANG OS Questions](03_faang_questions.md)**
   - Google: Design a file system
   - Amazon: Memory leak debugging
   - Microsoft: Thread synchronization
   - Meta: Process scheduling
   - Apple: Kernel development
   - Company-specific patterns

4. **[OS Coding Problems](04_coding_problems.md)**
   - Implement LRU cache
   - Producer-consumer with semaphores
   - Reader-writer locks
   - Thread pool implementation
   - Memory allocator
   - Banker's algorithm implementation

## Study Strategy

### Week 1-2: Fundamentals
- Process vs thread
- Memory hierarchy
- System calls
- Context switching

### Week 3-4: Core Concepts
- CPU scheduling algorithms
- Page replacement algorithms
- Synchronization primitives
- Deadlock handling

### Week 5-6: Advanced & Practice
- Virtual memory deep dive
- File systems
- Practice coding problems
- Mock interviews

## Key Interview Tips

1. **Start with clarifying questions** - Understand what the interviewer is asking
2. **Explain at multiple levels** - High-level concept, then implementation details
3. **Draw diagrams** - Process states, page tables, resource allocation graphs
4. **Discuss trade-offs** - Every design decision has pros and cons
5. **Use real-world examples** - Linux, Windows, macOS implementations
6. **Write clean code** - For coding problems, use proper synchronization
7. **Test your solution** - Consider edge cases, race conditions, deadlocks

## Must-Know Topics

### Process Management (High Priority)
- Process lifecycle and states
- fork(), exec(), wait() system calls
- Process Control Block (PCB)
- Context switching overhead
- IPC mechanisms

### Thread Management (High Priority)
- Thread vs process
- Thread models (user-level, kernel-level)
- Thread synchronization
- Thread pools

### Memory Management (Very High Priority)
- Paging vs segmentation
- TLB and page tables
- Virtual memory
- Page replacement (LRU, FIFO, Optimal)
- Thrashing
- Memory allocation strategies

### Synchronization (Very High Priority)
- Critical section problem
- Mutex vs semaphore vs monitor
- Classic problems (Producer-Consumer, Readers-Writers, Dining Philosophers)
- Deadlock detection and prevention

### CPU Scheduling (Medium Priority)
- FCFS, SJF, Priority, Round Robin
- Multilevel feedback queue
- Real-time scheduling

### File Systems (Medium Priority)
- File allocation methods
- Directory structure
- Unix inode
- Journaling

### Deadlocks (High Priority)
- Four necessary conditions
- Prevention, avoidance, detection
- Banker's algorithm
- Resource allocation graph

## Common Interview Questions

### Conceptual
1. What happens when you type a URL and press enter? (OS perspective)
2. Explain the boot process
3. What is a system call? Give examples
4. Difference between process and thread?
5. What is virtual memory? Why is it useful?
6. Explain page faults and how they're handled
7. What is thrashing?
8. Compare mutex and semaphore
9. Explain the four conditions for deadlock
10. How does copy-on-write work?

### Algorithm/Problem Solving
1. Solve page replacement for a given reference string
2. Calculate average waiting time for CPU scheduling
3. Solve Banker's algorithm problem
4. Implement producer-consumer with semaphores
5. Design a thread-safe LRU cache
6. Detect deadlock from resource allocation graph
7. Implement reader-writer locks

### System Design
1. Design a file system
2. Design a memory allocator
3. Design a thread pool
4. How would you implement virtual memory?
5. Design a scheduler for a real-time OS

## Resources for Practice

- **LeetCode**: Concurrency problems (1115, 1116, 1195, etc.)
- **Operating System Concepts** (Dinosaur Book): Classic textbook
- **GeeksforGeeks**: OS interview questions
- **System design interviews**: "Designing Data-Intensive Applications"

---

Good luck with your interviews! Remember: understand the concepts deeply, practice coding problems, and communicate clearly.
