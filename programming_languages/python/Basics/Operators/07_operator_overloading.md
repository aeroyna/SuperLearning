# Operator Overloading

Python allows custom classes to define how operators work on their instances through special methods (dunder methods).

---

## 1. Arithmetic Operators

| Operator | Method | Reverse Method |
|----------|--------|----------------|
| `+` | `__add__` | `__radd__` |
| `-` | `__sub__` | `__rsub__` |
| `*` | `__mul__` | `__rmul__` |
| `/` | `__truediv__` | `__rtruediv__` |
| `//` | `__floordiv__` | `__rfloordiv__` |
| `%` | `__mod__` | `__rmod__` |
| `**` | `__pow__` | `__rpow__` |
| `@` | `__matmul__` | `__rmatmul__` |

### Example: Vector Class

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __repr__(self):
        return f"Vector({self.x}, {self.y})"

    def __add__(self, other):
        if isinstance(other, Vector):
            return Vector(self.x + other.x, self.y + other.y)
        return NotImplemented

    def __mul__(self, scalar):
        if isinstance(scalar, (int, float)):
            return Vector(self.x * scalar, self.y * scalar)
        return NotImplemented

    def __rmul__(self, scalar):
        return self.__mul__(scalar)  # Multiplication is commutative

v1 = Vector(1, 2)
v2 = Vector(3, 4)

v1 + v2      # Vector(4, 6)
v1 * 3       # Vector(3, 6)
3 * v1       # Vector(3, 6) — uses __rmul__
```

### Reverse Methods

Called when the left operand doesn't support the operation:

```python
3 * v1  # Python tries:
        # 1. (3).__mul__(v1) → NotImplemented
        # 2. v1.__rmul__(3) → Vector(3, 6)
```

---

## 2. Augmented Assignment Operators

| Operator | Method |
|----------|--------|
| `+=` | `__iadd__` |
| `-=` | `__isub__` |
| `*=` | `__imul__` |
| `/=` | `__itruediv__` |
| `//=` | `__ifloordiv__` |
| `%=` | `__imod__` |
| `**=` | `__ipow__` |

```python
class Vector:
    # ... previous code ...

    def __iadd__(self, other):
        if isinstance(other, Vector):
            self.x += other.x
            self.y += other.y
            return self  # Return self for in-place modification
        return NotImplemented

v = Vector(1, 2)
v += Vector(3, 4)  # v is now Vector(4, 6)
```

If `__iadd__` is not defined, Python falls back to `__add__` and assignment.

---

## 3. Comparison Operators

| Operator | Method |
|----------|--------|
| `==` | `__eq__` |
| `!=` | `__ne__` |
| `<` | `__lt__` |
| `<=` | `__le__` |
| `>` | `__gt__` |
| `>=` | `__ge__` |

```python
class Money:
    def __init__(self, amount, currency='USD'):
        self.amount = amount
        self.currency = currency

    def __eq__(self, other):
        if isinstance(other, Money):
            return self.amount == other.amount and self.currency == other.currency
        return NotImplemented

    def __lt__(self, other):
        if isinstance(other, Money):
            if self.currency != other.currency:
                raise ValueError("Cannot compare different currencies")
            return self.amount < other.amount
        return NotImplemented

m1 = Money(100)
m2 = Money(100)
m3 = Money(200)

m1 == m2  # True
m1 < m3   # True
```

### Using `functools.total_ordering`

Define just `__eq__` and one ordering method, get the rest for free:

```python
from functools import total_ordering

@total_ordering
class Money:
    def __init__(self, amount, currency='USD'):
        self.amount = amount
        self.currency = currency

    def __eq__(self, other):
        if isinstance(other, Money):
            return self.amount == other.amount and self.currency == other.currency
        return NotImplemented

    def __lt__(self, other):
        if isinstance(other, Money):
            return self.amount < other.amount
        return NotImplemented

# Now <=, >, >= all work automatically!
```

---

## 4. Unary Operators

| Operator | Method |
|----------|--------|
| `-` | `__neg__` |
| `+` | `__pos__` |
| `abs()` | `__abs__` |
| `~` | `__invert__` |

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __neg__(self):
        return Vector(-self.x, -self.y)

    def __abs__(self):
        return (self.x ** 2 + self.y ** 2) ** 0.5

v = Vector(3, 4)
-v       # Vector(-3, -4)
abs(v)   # 5.0
```

---

## 5. Bitwise Operators

| Operator | Method | Reverse Method |
|----------|--------|----------------|
| `&` | `__and__` | `__rand__` |
| `\|` | `__or__` | `__ror__` |
| `^` | `__xor__` | `__rxor__` |
| `~` | `__invert__` | — |
| `<<` | `__lshift__` | `__rlshift__` |
| `>>` | `__rshift__` | `__rrshift__` |

```python
class Flags:
    def __init__(self, value=0):
        self.value = value

    def __or__(self, other):
        if isinstance(other, Flags):
            return Flags(self.value | other.value)
        return NotImplemented

    def __and__(self, other):
        if isinstance(other, Flags):
            return Flags(self.value & other.value)
        return NotImplemented

READ = Flags(0b001)
WRITE = Flags(0b010)
EXECUTE = Flags(0b100)

perms = READ | WRITE  # Flags(0b011)
```

---

## 6. Container Operations

| Operation | Method |
|-----------|--------|
| `len(obj)` | `__len__` |
| `obj[key]` | `__getitem__` |
| `obj[key] = value` | `__setitem__` |
| `del obj[key]` | `__delitem__` |
| `key in obj` | `__contains__` |
| `iter(obj)` | `__iter__` |

```python
class Deck:
    def __init__(self):
        self.cards = [f"{rank}{suit}"
                      for suit in "♠♥♦♣"
                      for rank in "A23456789TJQK"]

    def __len__(self):
        return len(self.cards)

    def __getitem__(self, index):
        return self.cards[index]

    def __contains__(self, card):
        return card in self.cards

deck = Deck()
len(deck)        # 52
deck[0]          # 'A♠'
deck[-1]         # 'K♣'
deck[0:4]        # ['A♠', '2♠', '3♠', '4♠']
'A♠' in deck     # True
```

---

## 7. Type Conversion

| Function | Method |
|----------|--------|
| `int()` | `__int__` |
| `float()` | `__float__` |
| `complex()` | `__complex__` |
| `bool()` | `__bool__` |
| `str()` | `__str__` |
| `repr()` | `__repr__` |
| `bytes()` | `__bytes__` |
| `hash()` | `__hash__` |

```python
class Fraction:
    def __init__(self, num, den):
        self.num = num
        self.den = den

    def __float__(self):
        return self.num / self.den

    def __int__(self):
        return self.num // self.den

    def __bool__(self):
        return self.num != 0

    def __str__(self):
        return f"{self.num}/{self.den}"

    def __repr__(self):
        return f"Fraction({self.num}, {self.den})"

f = Fraction(3, 4)
float(f)  # 0.75
int(f)    # 0
bool(f)   # True
str(f)    # "3/4"
repr(f)   # "Fraction(3, 4)"
```

---

## 8. Callable Objects

```python
class Counter:
    def __init__(self):
        self.count = 0

    def __call__(self, increment=1):
        self.count += increment
        return self.count

counter = Counter()
counter()      # 1
counter()      # 2
counter(5)     # 7
```

---

## 9. Context Managers

```python
class Timer:
    def __enter__(self):
        import time
        self.start = time.time()
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        import time
        self.elapsed = time.time() - self.start
        print(f"Elapsed: {self.elapsed:.3f}s")
        return False  # Don't suppress exceptions

with Timer() as t:
    # Do something
    sum(range(1000000))
# Prints: Elapsed: 0.023s
```

---

## 10. Best Practices

### Return `NotImplemented`
```python
def __add__(self, other):
    if isinstance(other, MyClass):
        # ... do addition
        pass
    return NotImplemented  # Let Python try other.__radd__
```

### Be Consistent
```python
# If you define __eq__, consider __hash__
class Point:
    def __eq__(self, other):
        return (self.x, self.y) == (other.x, other.y)

    def __hash__(self):
        return hash((self.x, self.y))
```

### Keep Operations Logical
```python
# Good: Vector + Vector = Vector
# Good: Vector * scalar = Vector
# Questionable: Vector * Vector = ??? (dot product? cross product? element-wise?)
```

---

## 11. Practice Problems

1. Create a `Rational` class that supports `+`, `-`, `*`, `/` with automatic simplification.

2. Implement a `Matrix` class with `+`, `*`, `@` (matrix multiplication), and indexing.

3. Create a `Temperature` class that can be compared and converted between Celsius and Fahrenheit.

---

## Summary

Operator overloading makes custom classes behave like built-in types:

| Category | Key Methods |
|----------|-------------|
| Arithmetic | `__add__`, `__mul__`, etc. |
| Comparison | `__eq__`, `__lt__`, etc. |
| Container | `__len__`, `__getitem__`, `__contains__` |
| Type conversion | `__int__`, `__float__`, `__bool__`, `__str__` |
| Callable | `__call__` |
| Context manager | `__enter__`, `__exit__` |
