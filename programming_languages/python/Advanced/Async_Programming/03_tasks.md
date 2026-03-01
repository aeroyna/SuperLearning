# Tasks and Futures

## Coroutines vs Tasks
*   **Coroutine**: The function code itself (`async def`). Calling it returns a coroutine object but doesn't run it.
*   **Task**: A wrapper that schedules the coroutine on the event loop.

## Creating Tasks

```python
task = asyncio.create_task(coro())
```
This submits the coroutine to the loop immediately.

## Example: Timeouts

```python
async def eternity():
    await asyncio.sleep(3600)
    print('yay!')

async def main():
    try:
        await asyncio.wait_for(eternity(), timeout=1.0)
    except asyncio.TimeoutError:
        print('timeout!')

asyncio.run(main())
```

## Task Groups (Python 3.11+)
A modern, safer way to manage multiple tasks (replaces `gather` in many cases). It ensures if one task fails, others are cancelled (Structured Concurrency).

```python
async def main():
    async with asyncio.TaskGroup() as tg:
        task1 = tg.create_task(some_coro())
        task2 = tg.create_task(another_coro())
    
    # Implicit await here.
    # If tasks raise exceptions, they are combined into an ExceptionGroup
    print("All done")
```
