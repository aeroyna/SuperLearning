# Bitwise Operators

Bitwise operators work on the binary representation of integers.

---

## 1. Bitwise Operators Overview

| Operator | Name | Example | Result |
|----------|------|---------|--------|
| `&` | AND | `0b1100 & 0b1010` | `0b1000` (8) |
| `\|` | OR | `0b1100 \| 0b1010` | `0b1110` (14) |
| `^` | XOR | `0b1100 ^ 0b1010` | `0b0110` (6) |
| `~` | NOT | `~0b1100` | `-13` |
| `<<` | Left shift | `0b0011 << 2` | `0b1100` (12) |
| `>>` | Right shift | `0b1100 >> 2` | `0b0011` (3) |

---

## 2. Binary Representation

```python
# Binary literals
a = 0b1100  # 12
b = 0b1010  # 10

# View binary
bin(12)    # '0b1100'
bin(-12)   # '-0b1100'

# Format with fixed width
format(12, '08b')  # '00001100'
f"{12:08b}"        # '00001100'

# Parse binary string
int('1100', 2)  # 12
```

---

## 3. AND (`&`)

Sets bits to 1 only where **both** operands have 1:

```
  1100 (12)
& 1010 (10)
------
  1000 (8)
```

```python
12 & 10  # 8

# Common uses:
# Check if bit is set
n = 0b1101
n & 0b0100  # 4 (non-zero = bit 2 is set)
n & 0b0010  # 0 (zero = bit 1 is not set)

# Check if even/odd
n & 1  # 0 if even, 1 if odd

# Mask lower bits
n & 0xFF  # Keep only lowest 8 bits
```

---

## 4. OR (`|`)

Sets bits to 1 where **either** operand has 1:

```
  1100 (12)
| 1010 (10)
------
  1110 (14)
```

```python
12 | 10  # 14

# Common uses:
# Set a bit
n = 0b1000
n | 0b0010  # 0b1010 (set bit 1)

# Combine flags
READ = 0b001
WRITE = 0b010
EXECUTE = 0b100

permissions = READ | WRITE  # 0b011
```

---

## 5. XOR (`^`)

Sets bits to 1 where operands **differ**:

```
  1100 (12)
^ 1010 (10)
------
  0110 (6)
```

```python
12 ^ 10  # 6

# Useful properties:
a ^ 0 == a      # XOR with 0 = identity
a ^ a == 0      # XOR with self = 0
a ^ b ^ b == a  # XOR twice = original

# Common uses:
# Toggle bits
n = 0b1100
n ^ 0b0010  # 0b1110 (toggle bit 1)

# Swap without temp variable
a, b = 5, 3
a ^= b
b ^= a
a ^= b
print(a, b)  # 3, 5

# Find the single unique element
nums = [1, 2, 3, 2, 1]
result = 0
for n in nums:
    result ^= n
print(result)  # 3
```

---

## 6. NOT (`~`)

Inverts all bits (two's complement):

```python
~0   # -1
~1   # -2
~12  # -13

# Formula: ~n = -(n+1)
```

Two's complement representation:
```
 12 in binary: 00001100
~12 in binary: 11110011 (which is -13 in two's complement)
```

---

## 7. Left Shift (`<<`)

Shifts bits left, filling with zeros:

```python
3 << 1   # 6  (0b11 → 0b110)
3 << 2   # 12 (0b11 → 0b1100)
3 << 3   # 24 (0b11 → 0b11000)

# Left shift by n = multiply by 2^n
x << n == x * (2 ** n)

# Fast multiplication by powers of 2
5 << 3  # 40 = 5 * 8 = 5 * 2³
```

---

## 8. Right Shift (`>>`)

Shifts bits right, discarding overflow:

```python
12 >> 1  # 6  (0b1100 → 0b110)
12 >> 2  # 3  (0b1100 → 0b11)
12 >> 3  # 1  (0b1100 → 0b1)
12 >> 4  # 0  (0b1100 → 0b0)

# Right shift by n = divide by 2^n (floor)
x >> n == x // (2 ** n)

# Fast division by powers of 2
40 >> 3  # 5 = 40 // 8 = 40 // 2³

# Negative numbers: arithmetic shift (preserves sign)
-8 >> 1  # -4 (not 4!)
```

---

## 9. Bit Manipulation Techniques

### Check if Bit is Set
```python
def is_bit_set(n, pos):
    return (n & (1 << pos)) != 0

is_bit_set(0b1010, 1)  # True (bit 1 is set)
is_bit_set(0b1010, 2)  # False (bit 2 is not set)
```

### Set a Bit
```python
def set_bit(n, pos):
    return n | (1 << pos)

set_bit(0b1000, 1)  # 0b1010
```

### Clear a Bit
```python
def clear_bit(n, pos):
    return n & ~(1 << pos)

clear_bit(0b1010, 1)  # 0b1000
```

### Toggle a Bit
```python
def toggle_bit(n, pos):
    return n ^ (1 << pos)

toggle_bit(0b1010, 1)  # 0b1000
toggle_bit(0b1000, 1)  # 0b1010
```

### Count Set Bits (Population Count)
```python
def count_bits(n):
    count = 0
    while n:
        count += n & 1
        n >>= 1
    return count

# Or use built-in
bin(12).count('1')  # 2
(12).bit_count()    # 2 (Python 3.10+)
```

### Check if Power of 2
```python
def is_power_of_2(n):
    return n > 0 and (n & (n - 1)) == 0

is_power_of_2(8)   # True (0b1000 & 0b0111 = 0)
is_power_of_2(10)  # False (0b1010 & 0b1001 ≠ 0)
```

### Get Lowest Set Bit
```python
def lowest_set_bit(n):
    return n & (-n)

lowest_set_bit(0b1010)  # 0b10 = 2
lowest_set_bit(0b1100)  # 0b100 = 4
```

---

## 10. Practical Applications

### Permissions/Flags
```python
# Define flags
NONE = 0b0000
READ = 0b0001
WRITE = 0b0010
EXECUTE = 0b0100
DELETE = 0b1000

# Combine permissions
user_perms = READ | WRITE  # 0b0011

# Check permission
if user_perms & READ:
    print("Can read")

# Add permission
user_perms |= EXECUTE

# Remove permission
user_perms &= ~WRITE

# Check multiple permissions
required = READ | WRITE
if (user_perms & required) == required:
    print("Has all required permissions")
```

### Color Manipulation (RGB)
```python
# Pack RGB into single integer
def rgb_to_int(r, g, b):
    return (r << 16) | (g << 8) | b

# Unpack integer to RGB
def int_to_rgb(color):
    r = (color >> 16) & 0xFF
    g = (color >> 8) & 0xFF
    b = color & 0xFF
    return r, g, b

color = rgb_to_int(255, 128, 64)  # 0xFF8040
print(int_to_rgb(color))  # (255, 128, 64)
```

### IP Address Manipulation
```python
# IP to integer
def ip_to_int(ip):
    parts = [int(p) for p in ip.split('.')]
    return (parts[0] << 24) | (parts[1] << 16) | (parts[2] << 8) | parts[3]

# Integer to IP
def int_to_ip(n):
    return f"{(n >> 24) & 0xFF}.{(n >> 16) & 0xFF}.{(n >> 8) & 0xFF}.{n & 0xFF}"

ip_to_int("192.168.1.1")  # 3232235777
int_to_ip(3232235777)     # "192.168.1.1"
```

---

## 11. Practice Problems

1. What's the output?
```python
print(0b1111 & 0b1010)
print(0b1111 ^ 0b1010)
print(1 << 10)
```

2. Write a function to swap two numbers using XOR without a temp variable.

3. Given an array where every element appears twice except one, find the unique element using XOR.

---

## Next Steps
- [Assignment Operators](05_assignment_operators.md)
