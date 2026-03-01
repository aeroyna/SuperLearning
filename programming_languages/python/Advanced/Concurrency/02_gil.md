# The Global Interpreter Lock (GIL)

The GIL is a mutex that protects access to Python objects, preventing multiple native threads from executing Python bytecodes at once.

## Why does it exist?
Python's memory management relies on **Reference Counting**. This is not thread-safe. If two threads decrement a reference count simultaneously, memory leaks or double-frees could occur.
Protecting every single object with a lock would be too slow and deadlock-prone. The GIL is a coarse-grained lock on the interpreter itself.

## Impact

### 1. CPU-Bound Tasks (Bad)
Code that performs heavy calculation (e.g., matrix multiplication, image processing) will **not** run faster with threads. In fact, it might run slower due to context switching overhead. Only one thread runs at a time.

### 2. I/O-Bound Tasks (Good)
Code that waits for external events (Network requests, Disk I/O).
When a thread waits for I/O, it **releases the GIL**, allowing other threads to run. This makes threading efficient for servers or web scrapers.

### 3. C Extensions (Good)
Libraries like `numpy` releases the GIL when doing heavy number crunching in C. This allows true parallelism even in Python scripts.

## Workarounds
1.  **Multiprocessing**: Use separate processes (each has its own GIL).
2.  **C Extensions**: Write critical paths in C/C++/Rust.
3.  **Python 3.12+**: Experimental support for per-interpreter GILs.
