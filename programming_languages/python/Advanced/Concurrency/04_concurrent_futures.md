# Concurrent Futures

The `concurrent.futures` module provides a high-level interface for asynchronously executing callables. It abstracts away the complexity of `threading` and `multiprocessing`.

## Executor Objects

### ThreadPoolExecutor
For I/O bound tasks.

```python
from concurrent.futures import ThreadPoolExecutor
import time

def task(n):
    time.sleep(1)
    return n * n

with ThreadPoolExecutor(max_workers=3) as executor:
    # Submit returns a Future object immediately
    future = executor.submit(task, 5)
    print(future.result()) # Blocks until done

    # Map applies function to iterable
    results = executor.map(task, [1, 2, 3])
    print(list(results))
```

### ProcessPoolExecutor
For CPU bound tasks.

```python
from concurrent.futures import ProcessPoolExecutor

def heavy_task(n):
    return sum(range(n))

if __name__ == '__main__':
    with ProcessPoolExecutor() as executor:
        results = executor.map(heavy_task, [10**6, 10**7])
```

## Futures
A `Future` represents an eventual result of an asynchronous operation.
*   `done()`: True if completed.
*   `result(timeout)`: Return value (or raise exception).
*   `add_done_callback(fn)`: Run fn when finished.

## When to use what?
*   **Threads (I/O)**: `concurrent.futures` is usually cleaner than the `threading` module unless you need low-level locking.
*   **Processes (CPU)**: `ProcessPoolExecutor` is simpler than `multiprocessing.Pool` for simple map/submit jobs.
