# Generators

Generators are a simple way to create iterators. They are functions that "pause" execution and resume where they left off.

## The `yield` Keyword

When a function contains `yield`, it becomes a generator function. Calling it returns a **generator object**; it does *not* execute the code immediately.

```python
def count_up_to(max):
    count = 1
    while count <= max:
        yield count
        count += 1

counter = count_up_to(3)
print(counter) # <generator object ...>

print(next(counter)) # 1
print(next(counter)) # 2
print(next(counter)) # 3
# print(next(counter)) # StopIteration
```

## How it Works
1.  **Call**: Returns generator object. Code execution hasn't started.
2.  **first `next()`**: Runs code until it hits `yield`. Returns the yielded value. **Pauses**.
3.  **Local State**: Python saves variable state (stack frame).
4.  **next `next()`**: Resumes exactly where it left off (after the `yield`).

## Benefits
*   **Memory Efficiency**: Generates values one at a time. A list of 1M integers takes ~35MB; a generator takes bytes.
*   **Infinite Streams**: Can represent infinite sequences (e.g., Fibonacci).
*   **Composable**: Can be piped into each other.
