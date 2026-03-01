# Async Patterns

## 1. Producer-Consumer
Using `asyncio.Queue` to distribute work.

```python
import asyncio
import random

async def producer(queue):
    for i in range(5):
        await asyncio.sleep(random.random())
        await queue.put(i)
        print(f'Produced {i}')
    await queue.put(None) # Sentinel

async def consumer(queue):
    while True:
        item = await queue.get()
        if item is None: break
        print(f'Consumed {item}')
        queue.task_done()

async def main():
    queue = asyncio.Queue()
    await asyncio.gather(producer(queue), consumer(queue))

asyncio.run(main())
```

## 2. Semaphore (Limiting Concurrency)
Limit the number of concurrent requests (e.g., to respect API rate limits).

```python
sem = asyncio.Semaphore(10) # Max 10 concurrent

async def limited_fetch(url):
    async with sem:
        return await fetch(url)
```

## 3. Async Generators
Iterate asynchronously.

```python
async def ticker(delay, to):
    for i in range(to):
        yield i
        await asyncio.sleep(delay)

async def main():
    async for i in ticker(1, 3):
        print(i)
```

## 4. Async Context Managers

```python
class AsyncContext:
    async def __aenter__(self):
        await setup()
        return self

    async def __aexit__(self, exc_type, exc, tb):
        await teardown()

async with AsyncContext() as ctx:
    ...
```
