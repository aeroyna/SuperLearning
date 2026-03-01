# Top 50 OS Interview Questions\n\n## Top 50 OS Interview Questions

### Most asked questions with answers

## Top Questions

### 1. Process vs Thread
**Answer**: Process is independent execution unit with own memory space. Thread shares memory with other threads in same process.

### 2. What is a deadlock?
**Answer**: Situation where processes wait indefinitely for resources held by each other. Requires: mutual exclusion, hold and wait, no preemption, circular wait.

### 3. Virtual Memory
**Answer**: Technique allowing execution of processes not completely in memory. Uses demand paging to load pages as needed.

### 4. Page Replacement Algorithms
**Answer**: 
- **FIFO**: Replace oldest page
- **LRU**: Replace least recently used
- **Optimal**: Replace page not used for longest time (theoretical)

### 5. Critical Section Problem
**Answer**: Ensure mutual exclusion when accessing shared resources. Requirements: mutual exclusion, progress, bounded waiting.

## Code Examples

### Producer-Consumer
```c
sem_t empty, full;
pthread_mutex_t mutex;

void *producer(void *arg) {
    while(1) {
        sem_wait(&empty);
        pthread_mutex_lock(&mutex);
        // Produce item
        pthread_mutex_unlock(&mutex);
        sem_post(&full);
    }
}

void *consumer(void *arg) {
    while(1) {
        sem_wait(&full);
        pthread_mutex_lock(&mutex);
        // Consume item
        pthread_mutex_unlock(&mutex);
        sem_post(&empty);
    }
}
```

### Readers-Writers
```java
class ReadWriteLock {
    private int readers = 0;
    private boolean writer = false;
    
    synchronized void acquireRead() {
        while (writer) wait();
        readers++;
    }
    
    synchronized void releaseRead() {
        readers--;
        notifyAll();
    }
    
    synchronized void acquireWrite() {
        while (readers > 0 || writer) wait();
        writer = true;
    }
    
    synchronized void releaseWrite() {
        writer = false;
        notifyAll();
    }
}
```

## Study Tips

1. **Understand fundamentals**: Don't just memorize
2. **Draw diagrams**: State diagrams, resource allocation graphs
3. **Practice coding**: Implement synchronization primitives
4. **Know trade-offs**: Every design has pros and cons
5. **Real-world examples**: Linux, Windows implementations

## Key Focus Areas for FAANG

- **Process scheduling**: Algorithms and implementation
- **Memory management**: Paging, segmentation, virtual memory
- **Synchronization**: Mutexes, semaphores, monitors
- **Deadlocks**: Detection, prevention, avoidance
- **File systems**: Structure, allocation, caching

## Common Mistakes

1. **Confusing process and thread**
2. **Not understanding context switch overhead**
3. **Missing edge cases in synchronization**
4. **Incorrect deadlock analysis**
5. **Poor explanation of virtual memory**

## Practice Problems

1. Implement LRU cache
2. Solve dining philosophers problem
3. Implement thread-safe bounded buffer
4. Calculate average waiting time for scheduling algorithms
5. Design simple file system

## Resources

- Operating System Concepts (Silberschatz)
- Modern Operating Systems (Tanenbaum)
- Operating Systems: Three Easy Pieces (Free online)
- Linux kernel source code
- Practice on LeetCode concurrency problems

## Interview Strategy

1. **Clarify question**: Ask about constraints
2. **Discuss trade-offs**: Show understanding of design choices
3. **Write clean code**: Even in pseudocode
4. **Test your solution**: Walk through examples
5. **Optimize if asked**: Discuss time/space improvements

## Key Takeaways

1. Master fundamental concepts thoroughly
2. Practice coding synchronization problems
3. Understand real-world OS implementations
4. Be able to explain trade-offs clearly
5. Know current trends (containers, cloud, etc.)

## Final Tips

- Review this entire operating systems module
- Focus on topics in your job description
- Practice explaining concepts clearly
- Code common problems by hand
- Study specific company's technology stack
