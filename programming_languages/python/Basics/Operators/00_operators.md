# Operators in Python

Operators are special symbols that perform operations on operands. Python provides a rich set of operators for arithmetic, comparison, logic, and more.

---

## Overview

| Topic | Description |
|-------|-------------|
| [**1. Arithmetic Operators**](01_arithmetic_operators.md) | Math operations (+, -, *, /, //, %, **) |
| [**2. Comparison Operators**](02_comparison_operators.md) | Value comparison (==, !=, <, >, <=, >=) |
| [**3. Logical Operators**](03_logical_operators.md) | Boolean logic (and, or, not) |
| [**4. Bitwise Operators**](04_bitwise_operators.md) | Bit manipulation (&, \|, ^, ~, <<, >>) |
| [**5. Assignment Operators**](05_assignment_operators.md) | Variable assignment (=, +=, -=, etc.) |
| [**6. Special Operators**](06_special_operators.md) | Identity, membership, and walrus |
| [**7. Operator Overloading**](07_operator_overloading.md) | Custom operators for classes |

---

## Operator Precedence

From highest to lowest precedence:

| Precedence | Operator | Description |
|------------|----------|-------------|
| 1 | `()` | Parentheses |
| 2 | `**` | Exponentiation |
| 3 | `+x`, `-x`, `~x` | Unary plus, minus, NOT |
| 4 | `*`, `/`, `//`, `%` | Multiplication, division, modulo |
| 5 | `+`, `-` | Addition, subtraction |
| 6 | `<<`, `>>` | Bitwise shifts |
| 7 | `&` | Bitwise AND |
| 8 | `^` | Bitwise XOR |
| 9 | `\|` | Bitwise OR |
| 10 | `==`, `!=`, `<`, `<=`, `>`, `>=`, `is`, `is not`, `in`, `not in` | Comparisons |
| 11 | `not` | Boolean NOT |
| 12 | `and` | Boolean AND |
| 13 | `or` | Boolean OR |
| 14 | `:=` | Walrus operator |

```python
# Precedence example
2 + 3 * 4      # 14 (not 20)
2 ** 3 ** 2    # 512 (right-associative: 2 ** 9)
-2 ** 2        # -4 (not 4, exponentiation binds tighter)
(-2) ** 2      # 4 (use parentheses to override)
```

---

## Quick Reference

### Arithmetic
```python
10 + 3   # 13  Addition
10 - 3   # 7   Subtraction
10 * 3   # 30  Multiplication
10 / 3   # 3.333...  True division
10 // 3  # 3   Floor division
10 % 3   # 1   Modulo
10 ** 3  # 1000  Exponentiation
```

### Comparison
```python
5 == 5   # True   Equal
5 != 3   # True   Not equal
5 < 10   # True   Less than
5 > 10   # False  Greater than
5 <= 5   # True   Less or equal
5 >= 3   # True   Greater or equal

# Chained comparison
1 < 5 < 10  # True
```

### Logical
```python
True and False  # False
True or False   # True
not True        # False
```

### Identity and Membership
```python
a is b        # True if same object
a is not b    # True if different objects
x in [1,2,3]  # True if x is in list
x not in [1,2,3]  # True if x is not in list
```

---

## Next Steps
Start with [Arithmetic Operators](01_arithmetic_operators.md).
