# Thread Concept\n\n## Definition

A **thread** is the smallest unit of CPU utilization. Threads within the same process share code, data, and files but have individual stacks and registers.

## Process vs Thread

Single-threaded: One execution path
Multi-threaded: Multiple concurrent execution paths in same process

Benefits:
1. **Responsiveness**: App can continue if part blocked
2. **Resource Sharing**: Threads share memory (efficient)
3. **Economy**: Cheaper than process creation/switching
4. **Scalability**: Can leverage multiple CPU cores

##Implementation

**C (Pthreads)**:
```c
#include <pthread.h>

void *worker(void *arg) {
    printf("Thread running\\n");
    return NULL;
}

int main() {
    pthread_t thread;
    pthread_create(&thread, NULL, worker, NULL);
    pthread_join(thread, NULL);
    return 0;
}
```

**Python**:
```python
import threading

def worker():
    print("Thread running")

t = threading.Thread(target=worker)
t.start()
t.join()
```

**Java**:
```java
class MyThread extends Thread {
    public void run() {
        System.out.println("Thread running");
    }
}

MyThread t = new MyThread();
t.start();
t.join();
```

## Key Takeaways

1. Threads share code, data, files (separate stack, registers)
2. Faster creation and switching than processes
3. Enable parallelism on multicore systems
4. Require synchronization for shared data access

## Interview Focus

1. What is a thread? Difference from process?
2. Benefits of multithreading?
3. What do threads share vs not share?
4. Thread lifecycle states?
