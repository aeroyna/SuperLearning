# Common Descriptors

You use descriptors every day without realizing it.

## 1. Properties
`@property` creates a data descriptor that delegates access to the getter/setter functions.

```python
# Roughly how property works
class Property:
    def __init__(self, fget=None, fset=None):
        self.fget = fget
        self.fset = fset

    def __get__(self, obj, objtype=None):
        return self.fget(obj)

    def __set__(self, obj, value):
        self.fset(obj, value)
```

## 2. Methods
Native functions are non-data descriptors. When accessed on an instance (`obj.method`), `__get__` is called, which returns a **bound method** object (a wrapper that holds both the function and the instance `self`).

```python
def my_func(self): pass

class A:
    f = my_func

a = A()
# a.f calls my_func.__get__(a, A) -> returns bound method
```

## 3. Static/Class Methods
Wrappers that change how `__get__` behaves.
*   `staticmethod`: Returns function as-is.
*   `classmethod`: Returns bound method bound to the *class*, not instance.
