# Threading

Threading allows concurrent execution within a single process. In Python, due to the **Global Interpreter Lock (GIL)**, threading is best suited for **I/O-bound** tasks (network, disk), not CPU-bound tasks.

## The `threading` Module

```python
import threading
import time

def worker(id):
    print(f"Worker {id} starting")
    time.sleep(1)
    print(f"Worker {id} finished")

# Create threads
threads = []
for i in range(5):
    t = threading.Thread(target=worker, args=(i,))
    threads.append(t)
    t.start()

# Wait for completion
for t in threads:
    t.join()

print("All done")
```

## Synchronization
Since threads share memory, you must prevent race conditions using locks.

### Locks

```python
lock = threading.Lock()
counter = 0

def increment():
    global counter
    with lock:
        # Critical section
        current = counter
        time.sleep(0.0001) # Simulate work / context switch
        counter = current + 1
```

### Other Primitives
*   `RLock`: Reentrant lock (same thread can acquire multiple times).
*   `Semaphore`: Limits access to N threads.
*   `Event`: One thread signals others.
*   `Barrier`: Waits for N threads to reach a point.

## Daemon Threads
Threads that run in the background and do not keep the program running if the main thread exits.
`t.daemon = True` or `threading.Thread(..., daemon=True)`
