# Concurrency and Multithreading

Concurrency allows programs to make progress on multiple tasks. Python offers threading, multiprocessing, and async approaches.

---

## Overview

| Topic | Description |
|-------|-------------|
| [**1. Threading**](01_threading.md) | Concurrent execution with threads |
| [**2. The GIL**](02_gil.md) | Global Interpreter Lock explained |
| [**3. Multiprocessing**](03_multiprocessing.md) | True parallelism with processes |
| [**4. Concurrent Futures**](04_concurrent_futures.md) | High-level interface |

---

## Quick Reference

### Threading
```python
import threading

def worker(name):
    print(f"Worker {name} starting")
    time.sleep(1)
    print(f"Worker {name} done")

threads = []
for i in range(3):
    t = threading.Thread(target=worker, args=(i,))
    threads.append(t)
    t.start()

for t in threads:
    t.join()  # Wait for completion
```

### Multiprocessing
```python
from multiprocessing import Process, Pool

def square(x):
    return x ** 2

# Process
p = Process(target=square, args=(5,))
p.start()
p.join()

# Pool
with Pool(4) as pool:
    results = pool.map(square, range(10))
```

### concurrent.futures
```python
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor

def task(n):
    return n ** 2

# Thread pool
with ThreadPoolExecutor(max_workers=4) as executor:
    futures = [executor.submit(task, i) for i in range(10)]
    results = [f.result() for f in futures]

# Process pool
with ProcessPoolExecutor(max_workers=4) as executor:
    results = list(executor.map(task, range(10)))
```

---

## The GIL

The Global Interpreter Lock (GIL) is a mutex that protects Python objects:

- **Threading is good for**: I/O-bound tasks (file ops, network)
- **Multiprocessing is good for**: CPU-bound tasks (calculations)

```python
# I/O bound: threading works well
def download(url):
    return requests.get(url).text

with ThreadPoolExecutor() as executor:
    results = list(executor.map(download, urls))

# CPU bound: use multiprocessing
def heavy_computation(n):
    return sum(i * i for i in range(n))

with ProcessPoolExecutor() as executor:
    results = list(executor.map(heavy_computation, numbers))
```

---

## Synchronization

### Lock
```python
lock = threading.Lock()

with lock:
    # Critical section
    shared_resource += 1
```

### Queue
```python
from queue import Queue

q = Queue()
q.put(item)
item = q.get()
```

---

## When to Use What

| Scenario | Best Approach |
|----------|---------------|
| I/O-bound (network, files) | Threading or asyncio |
| CPU-bound (calculations) | Multiprocessing |
| Many I/O operations | asyncio |
| Simple parallelism | concurrent.futures |

---

## Next Steps
Start with [Threading](01_threading.md).
