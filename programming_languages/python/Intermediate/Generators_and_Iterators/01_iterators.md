# Iterators

Iteration is a fundamental concept in Python. Understanding the difference between an **iterable** and an **iterator** is key.

## Definitions

### Iterable
An object capable of returning its members one at a time.
*   Examples: `list`, `str`, `tuple`, `dict`, `range`.
*   Protocol: Implements `__iter__()`.

### Iterator
An object representing a stream of data.
*   Protocol: Implements `__iter__()` (returning self) AND `__next__()` (returning data or raising `StopIteration`).

## The Iteration Protocol

When you write `for x in my_list`, Python actually does this:

```python
my_list = [1, 2, 3]

# 1. Get iterator
iterator = iter(my_list)  # Calls my_list.__iter__()

while True:
    try:
        # 2. Get next item
        item = next(iterator) # Calls iterator.__next__()
        # 3. Process item
        print(item)
    except StopIteration:
        # 4. Stop loop
        break
```

## Creating a Custom Iterator

```python
class CountDown:
    def __init__(self, start):
        self.start = start

    def __iter__(self):
        return self

    def __next__(self):
        if self.start <= 0:
            raise StopIteration
        current = self.start
        self.start -= 1
        return current

for num in CountDown(3):
    print(num)
# 3, 2, 1
```
