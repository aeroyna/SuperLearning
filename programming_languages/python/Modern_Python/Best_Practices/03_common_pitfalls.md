# Common Pitfalls

Even experienced developers fall for these traps.

## 1. Mutable Default Arguments
**The Trap**: Default arguments are evaluated **once** when the function is defined, not when called.

```python
def append_to(num, target=[]):
    target.append(num)
    return target

print(append_to(1)) # [1]
print(append_to(2)) # [1, 2] !!! Shared list
```

**The Fix**: Use `None` as sentinel.
```python
def append_to(num, target=None):
    if target is None:
        target = []
    target.append(num)
    return target
```

## 2. Late Binding Closures
**The Trap**: Closures bind to variables, not values.

```python
funcs = [lambda: i for i in range(3)]
print([f() for f in funcs]) # [2, 2, 2] !!!
```
All lambdas point to the same `i`, which ended at 2.

**The Fix**: Force early binding with default arg.
```python
funcs = [lambda i=i: i for i in range(3)]
print([f() for f in funcs]) # [0, 1, 2]
```

## 3. Modifying List While Iterating
**The Trap**:
```python
nums = [1, 2, 3, 4]
for n in nums:
    if n % 2 == 0:
        nums.remove(n)
# Result: [1, 3, 4] (Skipped 4 because index shifted)
```

**The Fix**: Iterate over a copy.
```python
for n in nums[:]:
# OR
nums = [n for n in nums if n % 2 != 0]
```
