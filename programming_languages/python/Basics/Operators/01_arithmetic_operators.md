# Arithmetic Operators

## 1. Basic Arithmetic

| Operator | Name | Example | Result |
|----------|------|---------|--------|
| `+` | Addition | `10 + 3` | `13` |
| `-` | Subtraction | `10 - 3` | `7` |
| `*` | Multiplication | `10 * 3` | `30` |
| `/` | True Division | `10 / 3` | `3.333...` |
| `//` | Floor Division | `10 // 3` | `3` |
| `%` | Modulo | `10 % 3` | `1` |
| `**` | Exponentiation | `2 ** 10` | `1024` |

---

## 2. Division Types

### True Division (`/`)
Always returns a float, even for integers:

```python
10 / 2   # 5.0 (not 5)
10 / 3   # 3.3333333333333335
-10 / 3  # -3.3333333333333335
```

### Floor Division (`//`)
Returns the floor (rounds toward negative infinity):

```python
10 // 3   # 3
-10 // 3  # -4 (not -3! Rounds toward -∞)
10.0 // 3 # 3.0 (float if either operand is float)

# Compare with int() truncation
int(10 / 3)   # 3
int(-10 / 3)  # -3 (truncates toward zero)
```

### Modulo (`%`)
Returns the remainder, sign matches divisor:

```python
10 % 3    # 1
-10 % 3   # 2 (not -1! Sign matches divisor)
10 % -3   # -2

# Relationship: a = (a // b) * b + (a % b)
a, b = -10, 3
(a // b) * b + (a % b)  # -10 ✓
```

### divmod() Function
Returns both quotient and remainder:

```python
divmod(10, 3)   # (3, 1)
divmod(-10, 3)  # (-4, 2)

q, r = divmod(17, 5)
print(f"{17} = {q} * 5 + {r}")  # 17 = 3 * 5 + 2
```

---

## 3. Exponentiation

```python
2 ** 10      # 1024
2 ** 0.5     # 1.414... (square root)
(-1) ** 0.5  # (6.12...e-17+1j) — complex!
8 ** (1/3)   # 2.0 (cube root)

# Right-associative
2 ** 3 ** 2  # 512 = 2 ** 9 (not 8 ** 2)

# Use pow() for modular exponentiation
pow(2, 10)       # 1024
pow(2, 10, 100)  # 24 = (2 ** 10) % 100 — efficient!
```

---

## 4. Unary Operators

```python
x = 5
+x  # 5 (positive, rarely used)
-x  # -5 (negation)

# Double negative
--x  # 5 (not decrement like C!)
```

---

## 5. Type Coercion in Arithmetic

Python promotes to the "widest" type:

```python
# int + float → float
3 + 4.5   # 7.5

# int + complex → complex
3 + 4j    # (3+4j)

# float + complex → complex
3.5 + 4j  # (3.5+4j)

# bool is treated as int
True + True   # 2
False * 10    # 0
```

---

## 6. Numeric Edge Cases

### Infinity and NaN
```python
float('inf') + 1      # inf
float('inf') - float('inf')  # nan
float('inf') * 0      # nan (not 0!)

import math
math.isnan(float('nan'))  # True
math.isinf(float('inf'))  # True
```

### Large Numbers
```python
# Integers: unlimited precision
10 ** 100  # Works!

# Floats: overflow to infinity
1e308 * 10  # inf
```

### Precision Issues
```python
0.1 + 0.2  # 0.30000000000000004

# Use decimal for exact arithmetic
from decimal import Decimal
Decimal('0.1') + Decimal('0.2')  # Decimal('0.3')
```

---

## 7. Practical Examples

### Check if Number is Even/Odd
```python
def is_even(n):
    return n % 2 == 0

def is_odd(n):
    return n % 2 != 0
```

### Extract Digits
```python
n = 12345
ones = n % 10        # 5
tens = (n // 10) % 10  # 4
hundreds = (n // 100) % 10  # 3
```

### Time Conversion
```python
total_seconds = 3661

hours = total_seconds // 3600      # 1
minutes = (total_seconds % 3600) // 60  # 1
seconds = total_seconds % 60       # 1

print(f"{hours}:{minutes:02d}:{seconds:02d}")  # 1:01:01
```

### Circular Index
```python
items = ['a', 'b', 'c', 'd']
for i in range(10):
    print(items[i % len(items)], end=' ')
# a b c d a b c d a b
```

---

## Next Steps
- [Comparison Operators](02_comparison_operators.md)
