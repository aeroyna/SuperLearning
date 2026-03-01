# Advanced Generators

Generators are not just data producers; they can also be consumers (coroutines).

## `yield from` (Python 3.3+)
Delegates part of its operation to another generator. It simplifies nested generators and handles `send`/`throw` forwarding automatically.

```python
def sub_gen():
    yield 'A'
    yield 'B'

def main_gen():
    yield 'Start'
    yield from sub_gen()
    yield 'End'

print(list(main_gen())) # ['Start', 'A', 'B', 'End']
```

## `send()`
You can inject values back into the generator function. The `yield` expression returns the sent value.

```python
def accumulator():
    total = 0
    while True:
        value = yield total
        if value is None: break
        total += value

gen = accumulator()
print(next(gen))      # 0 (Must prime it)
print(gen.send(10))   # 10
print(gen.send(20))   # 30
```

## `throw()`
Injects an exception into the generator at the pause point.

```python
gen.throw(ValueError("Something went wrong"))
```

## `close()`
Stops the generator (raises `GeneratorExit` inside it).

```python
gen.close()
```
