# Multiprocessing

The `multiprocessing` module spawns processes instead of threads. Each process has its own Python interpreter and memory space, bypassing the GIL.

## Basic Usage

The API is very similar to `threading`.

```python
import multiprocessing
import time

def heavy_computation(x):
    return x * x

if __name__ == '__main__':
    # Essential on Windows/macOS to avoid recursive spawning
    p = multiprocessing.Process(target=heavy_computation, args=(10,))
    p.start()
    p.join()
```

## Sharing Data
Since memory is not shared, you must use Inter-Process Communication (IPC).

### 1. Queue
Thread/Process safe FIFO queue.

```python
def worker(q):
    q.put("Result")

q = multiprocessing.Queue()
p = multiprocessing.Process(target=worker, args=(q,))
p.start()
print(q.get()) # "Result"
```

### 2. Pipe
Two-way connection between two processes.

### 3. Shared Memory
`Value` or `Array` maps memory into multiple processes (faster but riskier).

```python
from multiprocessing import Value

count = Value('i', 0) # 'i' for signed int
```

## Pool
Manage a pool of worker processes.

```python
from multiprocessing import Pool

def square(x):
    return x*x

if __name__ == '__main__':
    with Pool(processes=4) as pool:
        print(pool.map(square, [1, 2, 3]))
        # [1, 4, 9]
```
