# Special Methods (Magic Methods)

Special methods, also known as **magic methods** or **dunder methods** (double underscore), allow classes to define how objects behave with built-in operations (arithmetic, iteration, comparisons, etc.).

## Core Special Methods

| Method | Trigger | Description |
|--------|---------|-------------|
| `__init__` | `obj = Class()` | Initialization |
| `__new__` | `obj = Class()` | Object creation (before init) |
| `__repr__` | `repr(obj)` | Official string representation (debug) |
| `__str__` | `str(obj)` | User-friendly string representation |
| `__call__` | `obj()` | Makes object callable like a function |

### `__repr__` vs `__str__`
*   `__repr__`: Should be unambiguous and, if possible, valid Python code to recreate the object.
*   `__str__`: Should be readable for end-users.

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __repr__(self):
        return f"Point({self.x}, {self.y})"

    def __str__(self):
        return f"({self.x}, {self.y})"
```

---

## Comparison Methods

| Method | Trigger |
|--------|---------|
| `__eq__` | `==` |
| `__ne__` | `!=` |
| `__lt__` | `<` |
| `__le__` | `<=` |
| `__gt__` | `>` |
| `__ge__` | `>=` |

By implementing these, you can sort your custom objects.

---

## Container Methods

To create a sequence or mapping (like a list or dict), implement these:

```python
class CustomList:
    def __init__(self, data):
        self._data = data

    def __len__(self):
        return len(self._data)

    def __getitem__(self, index):
        return self._data[index]

    def __setitem__(self, index, value):
        self._data[index] = value

    def __iter__(self):
        return iter(self._data)
```

Usage:
```python
cl = CustomList([1, 2, 3])
print(len(cl))  # 3
print(cl[0])    # 1
for item in cl: # Works via __iter__
    print(item)
```

---

## Context Managers

To support the `with` statement (cleanup logic):

```python
class ManagedResource:
    def __enter__(self):
        print("Acquiring resource")
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        print("Releasing resource")
        # Return True to suppress exceptions
```

---

## Attribute Access

| Method | Description |
|--------|-------------|
| `__getattr__` | Called when attribute lookup fails |
| `__getattribute__` | Called **every** time an attribute is accessed (dangerous!) |
| `__setattr__` | Called on attribute assignment |

```python
class Dynamic:
    def __getattr__(self, name):
        return f"You asked for {name}, but I don't have it."

d = Dynamic()
print(d.missing_attr)  # Works!
```

---

## Best Practices
1.  **Don't invent your own names**: Only use documented dunder methods.
2.  **Return `NotImplemented`**: In comparison/arithmetic methods, create a fallback mechanism by returning `NotImplemented` instead of raising explicit type errors.
