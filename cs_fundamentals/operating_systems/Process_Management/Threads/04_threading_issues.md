# Threading Issues\n\n## Major Threading Issues

### 1. Thread Cancellation

**Asynchronous**: Terminate immediately (dangerous)
**Deferred**: Thread checks cancellation points

```c
pthread_t thread;
pthread_create(&thread, NULL, work, NULL);
pthread_cancel(thread);  // Request cancellation
pthread_join(thread, NULL);
```

### 2. Signal Handling

Signals in multithreaded programs:
- Deliver to specific thread?
- Deliver to all threads?
- Deliver to certain threads?

### 3. Thread Pools

Pre-create threads to handle tasks efficiently.

Benefits:
- Faster than creating threads on-demand
- Limit on concurrent threads
- Separate task creation from execution

```java
ExecutorService pool = Executors.newFixedThreadPool(10);
pool.submit(() -> { /* task */ });
pool.shutdown();
```

### 4. Thread-Specific Data

Each thread needs own copy of certain data.

```c
pthread_key_t key;
pthread_key_create(&key, NULL);
pthread_setspecific(key, value);
void *data = pthread_getspecific(key);
```

### 5. Scheduler Activations

Communication between user-level and kernel-level thread schedulers.

## Race Conditions

```python
# Unsafe
counter = 0

def increment():
    global counter
    counter += 1  # Not atomic!

# Safe
import threading
lock = threading.Lock()

def increment():
    global counter
    with lock:
        counter += 1
```

## Deadlock

```c
// Thread 1
pthread_mutex_lock(&mutex1);
pthread_mutex_lock(&mutex2);

// Thread 2
pthread_mutex_lock(&mutex2);  // Deadlock!
pthread_mutex_lock(&mutex1);
```

**Prevention**: Always acquire locks in same order

## Key Takeaways

1. Thread cancellation can be asynchronous or deferred
2. Thread pools improve performance
3. Race conditions require synchronization
4. Deadlocks occur with circular lock dependencies
5. Thread-specific data provides per-thread storage

## Interview Focus

1. What is a race condition? How to prevent?
2. Explain thread pool benefits
3. How does thread cancellation work?
4. What causes deadlock? How to prevent?
5. When to use thread-specific data?
