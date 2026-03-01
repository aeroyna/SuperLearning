# Async Programming

Asyncio enables concurrent code using async/await syntax, ideal for I/O-bound and high-level structured network code.

---

## Overview

| Topic | Description |
|-------|-------------|
| [**1. Async Basics**](01_async_basics.md) | async/await, coroutines |
| [**2. Event Loop**](02_event_loop.md) | Running async code |
| [**3. Tasks and Gathering**](03_tasks.md) | Concurrent coroutines |
| [**4. Async Patterns**](04_async_patterns.md) | Common async idioms |

---

## Quick Reference

### Basic Coroutine
```python
import asyncio

async def fetch_data():
    print("Fetching...")
    await asyncio.sleep(1)  # Non-blocking wait
    print("Done!")
    return {"data": 42}

# Run coroutine
result = asyncio.run(fetch_data())
```

### Concurrent Execution
```python
async def fetch(url):
    await asyncio.sleep(1)  # Simulate network
    return f"Data from {url}"

async def main():
    # Run concurrently
    results = await asyncio.gather(
        fetch("url1"),
        fetch("url2"),
        fetch("url3")
    )
    return results

results = asyncio.run(main())
# Takes ~1 second, not 3
```

### Tasks
```python
async def main():
    # Create tasks for concurrent execution
    task1 = asyncio.create_task(fetch("url1"))
    task2 = asyncio.create_task(fetch("url2"))

    # Do other work
    print("Tasks running...")

    # Wait for completion
    result1 = await task1
    result2 = await task2
```

---

## Async Context Managers
```python
async with aiohttp.ClientSession() as session:
    async with session.get(url) as response:
        data = await response.text()
```

## Async Iterators
```python
async for item in async_generator():
    process(item)
```

---

## Common Libraries

| Library | Purpose |
|---------|---------|
| `aiohttp` | Async HTTP client/server |
| `aiofiles` | Async file operations |
| `asyncpg` | Async PostgreSQL |
| `aiomysql` | Async MySQL |
| `httpx` | Modern async HTTP |

---

## When to Use Asyncio

- Web servers handling many connections
- API clients making concurrent requests
- WebSocket applications
- Database applications with many queries

**Not ideal for**: CPU-bound tasks (use multiprocessing instead)

---

## Next Steps
Start with [Async Basics](01_async_basics.md).
