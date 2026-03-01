# Numeric Types in Python

Python provides three distinct numeric types: integers, floating-point numbers, and complex numbers.

---

## 1. Integers (`int`)

### Unlimited Precision
Python integers have **arbitrary precision**—they can be as large as memory allows:

```python
# Regular integer
x = 42

# Very large integer
big = 10 ** 100
print(len(str(big)))  # 101 digits!

# No overflow
import sys
print(sys.maxsize)     # Largest "native" int, but Python handles bigger
print(10 ** 1000)      # Works fine!
```

### Integer Literals
```python
# Decimal
x = 42

# Binary (0b prefix)
x = 0b101010  # 42

# Octal (0o prefix)
x = 0o52      # 42

# Hexadecimal (0x prefix)
x = 0x2a      # 42

# Underscores for readability (Python 3.6+)
million = 1_000_000
binary = 0b1111_0000_1111
```

### Integer Operations
```python
# Basic arithmetic
10 + 3   # 13 (addition)
10 - 3   # 7 (subtraction)
10 * 3   # 30 (multiplication)
10 / 3   # 3.333... (true division, returns float!)
10 // 3  # 3 (floor division, returns int)
10 % 3   # 1 (modulo)
10 ** 3  # 1000 (exponentiation)
-10      # -10 (negation)

# Bitwise operations
a, b = 0b1100, 0b1010  # 12, 10
a & b   # 0b1000 = 8 (AND)
a | b   # 0b1110 = 14 (OR)
a ^ b   # 0b0110 = 6 (XOR)
~a      # -13 (NOT, two's complement)
a << 2  # 0b110000 = 48 (left shift)
a >> 2  # 0b11 = 3 (right shift)
```

### Integer Methods
```python
x = 42

# Bit length
x.bit_length()  # 6 (bits needed to represent)

# Byte conversion
x.to_bytes(2, 'big')    # b'\x00*'
x.to_bytes(2, 'little') # b'*\x00'
int.from_bytes(b'\x00*', 'big')  # 42

# Check if integer (for float)
(4.0).is_integer()  # True
(4.5).is_integer()  # False
```

### CPython Integer Implementation
Small integers (-5 to 256) are cached and reused:

```python
# Same object (cached)
a = 100
b = 100
print(a is b)  # True

# Different objects (not cached)
a = 1000
b = 1000
print(a is b)  # False (usually)
```

Internally, large integers are stored as arrays of "digits":
```
PyLongObject:
┌─────────────────┐
│ ob_refcnt: 1    │  Reference count
│ ob_type: int    │  Type pointer
│ ob_size: 2      │  Number of digits
│ ob_digit[0]: X  │  Least significant digit
│ ob_digit[1]: Y  │  More significant digit
└─────────────────┘
```

---

## 2. Floating-Point (`float`)

### IEEE 754 Double Precision
Python floats are 64-bit IEEE 754 double-precision numbers:

```python
# Regular float
x = 3.14

# Scientific notation
x = 1.5e10   # 1.5 × 10^10
x = 1.5e-10  # 1.5 × 10^-10

# Special values
float('inf')   # Positive infinity
float('-inf')  # Negative infinity
float('nan')   # Not a Number
```

### Float Precision Issues
```python
# Classic precision problem
0.1 + 0.2  # 0.30000000000000004, not 0.3!

# Why? 0.1 cannot be exactly represented in binary
format(0.1, '.20f')  # '0.10000000000000000555'

# Use decimal for exact decimal arithmetic
from decimal import Decimal
Decimal('0.1') + Decimal('0.2')  # Decimal('0.3')

# Or round for display
round(0.1 + 0.2, 1)  # 0.3
```

### Float Range and Limits
```python
import sys

# Maximum float
sys.float_info.max  # 1.7976931348623157e+308

# Minimum positive float
sys.float_info.min  # 2.2250738585072014e-308

# Machine epsilon (smallest difference)
sys.float_info.epsilon  # 2.220446049250313e-16

# Overflow
1e308 * 10  # inf

# Underflow
1e-308 / 1e10  # 0.0 (or very small number)
```

### Float Methods
```python
x = 3.75

# Check for special values
import math
math.isnan(x)      # False
math.isinf(x)      # False
math.isfinite(x)   # True

# Get integer and fractional parts
math.modf(x)       # (0.75, 3.0)

# Ratio as fraction
x.as_integer_ratio()  # (15, 4) — 3.75 = 15/4

# Hex representation
x.hex()            # '0x1.e000000000000p+1'
float.fromhex('0x1.e000000000000p+1')  # 3.75
```

---

## 3. Complex Numbers (`complex`)

### Complex Literals
```python
# Using j suffix
z = 3 + 4j

# Using complex()
z = complex(3, 4)

# Real and imaginary parts
z.real  # 3.0
z.imag  # 4.0

# Conjugate
z.conjugate()  # (3-4j)
```

### Complex Operations
```python
z1 = 3 + 4j
z2 = 1 + 2j

z1 + z2   # (4+6j)
z1 - z2   # (2+2j)
z1 * z2   # (-5+10j)
z1 / z2   # (2.2-0.4j)

# Magnitude (absolute value)
abs(z1)   # 5.0 (sqrt(3² + 4²))

# Phase/angle (requires cmath)
import cmath
cmath.phase(z1)  # 0.9272... radians

# Polar coordinates
cmath.polar(z1)       # (5.0, 0.9272...) — (magnitude, phase)
cmath.rect(5, 0.927)  # ~(3+4j) — back to rectangular
```

---

## 4. Type Conversion

### Explicit Conversion
```python
# To int
int(3.7)      # 3 (truncates toward zero)
int(-3.7)     # -3
int("42")     # 42
int("2a", 16) # 42 (hex string)
int("101010", 2)  # 42 (binary string)

# To float
float(42)     # 42.0
float("3.14") # 3.14
float("inf")  # inf

# To complex
complex(3)       # (3+0j)
complex(3, 4)    # (3+4j)
complex("3+4j")  # (3+4j)
```

### Implicit Conversion (Coercion)
```python
# int + float → float
3 + 4.0    # 7.0

# int/float + complex → complex
3 + 4j     # (3+4j)
3.0 + 4j   # (3+4j)

# Hierarchy: complex > float > int
```

---

## 5. Decimal and Fraction (Standard Library)

### Decimal for Exact Decimal Arithmetic
```python
from decimal import Decimal, getcontext

# Exact representation
Decimal('0.1') + Decimal('0.2')  # Decimal('0.3')

# Set precision
getcontext().prec = 50
Decimal(1) / Decimal(7)  # 0.14285714285714285714285714285714285714285714285714

# Financial calculations
price = Decimal('19.99')
quantity = Decimal('3')
total = price * quantity  # Decimal('59.97')
```

### Fraction for Exact Rational Numbers
```python
from fractions import Fraction

# Create fractions
f1 = Fraction(1, 3)
f2 = Fraction(1, 6)

# Arithmetic
f1 + f2  # Fraction(1, 2)
f1 * f2  # Fraction(1, 18)

# From float (approximate)
Fraction(0.5)     # Fraction(1, 2)
Fraction('0.125') # Fraction(1, 8)

# Limit denominator
Fraction(3.14159).limit_denominator(100)  # Fraction(22, 7)
```

---

## 6. Math Module

```python
import math

# Constants
math.pi      # 3.141592653589793
math.e       # 2.718281828459045
math.tau     # 6.283185307179586 (2π)
math.inf     # Infinity

# Rounding
math.floor(3.7)   # 3
math.ceil(3.2)    # 4
math.trunc(-3.7)  # -3
round(3.5)        # 4 (banker's rounding in Python 3)

# Powers and logs
math.sqrt(16)     # 4.0
math.pow(2, 10)   # 1024.0
math.log(100, 10) # 2.0
math.log2(8)      # 3.0
math.log10(100)   # 2.0

# Trigonometry
math.sin(math.pi / 2)  # 1.0
math.cos(0)            # 1.0
math.tan(math.pi / 4)  # 0.999... (≈1)
math.degrees(math.pi)  # 180.0
math.radians(180)      # 3.14159...

# Combinatorics
math.factorial(5)      # 120
math.comb(5, 2)        # 10 (combinations)
math.perm(5, 2)        # 20 (permutations)
math.gcd(48, 18)       # 6
math.lcm(4, 6)         # 12 (Python 3.9+)
```

---

## 7. Random Numbers

```python
import random

# Random float [0.0, 1.0)
random.random()

# Random integer [a, b] inclusive
random.randint(1, 10)

# Random choice from sequence
random.choice(['a', 'b', 'c'])

# Multiple random choices
random.choices(['a', 'b', 'c'], k=5)  # With replacement
random.sample(['a', 'b', 'c'], k=2)   # Without replacement

# Shuffle in place
lst = [1, 2, 3, 4, 5]
random.shuffle(lst)

# Reproducible randomness
random.seed(42)
random.random()  # Always same result with same seed
```

---

## 8. Practice Problems

1. **Precision Check**: What is `0.1 + 0.1 + 0.1 - 0.3`? Why?

2. **Bit Manipulation**: Write a function to count the number of 1-bits in an integer.

3. **Temperature Converter**: Convert between Celsius and Fahrenheit using proper type handling.

4. **Complex Magnitude**: Write a function that returns whether a complex number is inside the unit circle.

---

## Next Steps
- [Boolean and None](03_boolean_and_none.md)
