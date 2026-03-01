# AsyncIO Basics

`asyncio` is a library to write concurrent code using the **async/await** syntax. It is used as a foundation for multiple Python asynchronous frameworks (e.g., FastAPI, Django 3+).

## Key Concepts
*   **Coroutines**: Functions defined with `async def`.
*   **Tasks**: Wrappers for coroutines to schedule them.
*   **Event Loop**: The engine that drives execution.

## Hello World

```python
import asyncio

async def main():
    print('Hello')
    await asyncio.sleep(1)
    print('World')

asyncio.run(main())
```

### The `await` keyword
`await` pauses the execution of the surrounding coroutine until the awaitable (coroutine, Task, or Future) completes. Critically, **it yields control back to the event loop**, allowing other tasks to run.

## Running Concurrently (`asyncio.gather`)

```python
import asyncio
import time

async def say_after(delay, what):
    await asyncio.sleep(delay)
    print(what)

async def main():
    print(f"started at {time.strftime('%X')}")

    # Run both concurrently
    await asyncio.gather(
        say_after(1, 'hello'),
        say_after(2, 'world')
    )

    print(f"finished at {time.strftime('%X')}")

asyncio.run(main())
```
**Result**: Takes ~2 seconds, not 3.
