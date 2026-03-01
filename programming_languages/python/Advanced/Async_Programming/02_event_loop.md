# The Event Loop

The event loop is the core of every asyncio application. It runs in a thread (usually the main thread) and executes all callbacks and tasks methods.

## How it works
1.  **Queue**: Keeps a list of scheduled tasks/callbacks.
2.  **Select**: Waits for I/O events (sockets readable/writable) using the OS selector (epoll/kqueue/IOCP).
3.  **Dispatch**: When an event occurs or a task is ready, it runs it.

> Unlike threading, where the OS preempts threads, asyncio is **cooperative multitasking**. A task must explicitly `await` to yield control. If a task blocks (e.g., `time.sleep` or heavy calculation), the entire loop freezes.

## Managing the Loop

`asyncio.run()` (Python 3.7+) handles creation and closing automatically.

### Manual Access
```python
loop = asyncio.get_running_loop()
```

### Blocking Code
To run blocking code (CPU bound or legacy I/O) without freezing the loop, use an executor.

```python
await loop.run_in_executor(None, blocking_function)
```

## uvloop
`uvloop` is a drop-in replacement for the standard asyncio event loop, written in Cython on top of libuv (same as Node.js). It makes asyncio 2-4x faster.

```python
import uvloop
# uvloop.install() # Pre-3.11
# In 3.11+, use asyncio.Runner(loop_factory=uvloop.new_event_loop)
```
