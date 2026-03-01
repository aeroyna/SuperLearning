## Common Bit Manipulation Tricks

Beyond the basic operators, there are several clever "tricks" or idiomatic patterns that use bit manipulation to solve problems efficiently. Knowing these can make certain problems much simpler.

### 1. Check if the i-th bit is set
To check if the `i`-th bit (from the right, 0-indexed) is set in a number `n`, you can `AND` it with a mask that has only the `i`-th bit set.

- **Trick**: `if (n & (1 << i)) != 0:`
- **Example**: To check if the 3rd bit of `n = 13 (1101)` is set:
  - `1 << 3` creates the mask `1000`.
  - `1101 & 1000 = 1000` (which is non-zero), so the bit is set.

>[!example]- C++
>```cpp
>bool isBitSet(int n, int i) {
>    return (n & (1 << i)) != 0;
>}
>
>// Example usage
>int n = 13;  // binary: 1101
>bool bit3 = isBitSet(n, 3);  // true (bit is set)
>bool bit1 = isBitSet(n, 1);  // false (bit is not set)
>```

>[!example]- Java
>```java
>public boolean isBitSet(int n, int i) {
>    return (n & (1 << i)) != 0;
>}
>
>// Example usage
>int n = 13;  // binary: 1101
>boolean bit3 = isBitSet(n, 3);  // true (bit is set)
>boolean bit1 = isBitSet(n, 1);  // false (bit is not set)
>```

>[!example]- Python
>```python
>def is_bit_set(n, i):
>    return (n & (1 << i)) != 0
>
># Example usage
>n = 13  # binary: 1101
>bit3 = is_bit_set(n, 3)  # True (bit is set)
>bit1 = is_bit_set(n, 1)  # False (bit is not set)
>```

>[!example]- JavaScript
>```javascript
>function isBitSet(n, i) {
>    return (n & (1 << i)) !== 0;
>}
>
>// Example usage
>const n = 13;  // binary: 1101
>const bit3 = isBitSet(n, 3);  // true (bit is set)
>const bit1 = isBitSet(n, 1);  // false (bit is not set)
>```

### 2. Set the i-th bit
To set the `i`-th bit of a number `n` to 1, you can `OR` it with a mask that has the `i`-th bit set.

- **Trick**: `n = n | (1 << i)`
- **Example**: To set the 2nd bit of `n = 10 (1010)`:
  - `1 << 2` creates the mask `0100`.
  - `1010 | 0100 = 1110` (which is 14).

>[!example]- C++
>```cpp
>int setBit(int n, int i) {
>    return n | (1 << i);
>}
>
>// Example usage
>int n = 10;  // binary: 1010
>int result = setBit(n, 2);  // result = 14, binary: 1110
>```

>[!example]- Java
>```java
>public int setBit(int n, int i) {
>    return n | (1 << i);
>}
>
>// Example usage
>int n = 10;  // binary: 1010
>int result = setBit(n, 2);  // result = 14, binary: 1110
>```

>[!example]- Python
>```python
>def set_bit(n, i):
>    return n | (1 << i)
>
># Example usage
>n = 10  # binary: 1010
>result = set_bit(n, 2)  # result = 14, binary: 1110
>```

>[!example]- JavaScript
>```javascript
>function setBit(n, i) {
>    return n | (1 << i);
>}
>
>// Example usage
>const n = 10;  // binary: 1010
>const result = setBit(n, 2);  // result = 14, binary: 1110
>```

### 3. Clear (or unset) the i-th bit
To clear the `i`-th bit of a number `n` (set it to 0), you can `AND` it with a mask that has every bit set to 1 *except* for the `i`-th bit. This mask is created by taking the `NOT` of the standard `1 << i` mask.

- **Trick**: `n = n & ~(1 << i)`
- **Example**: To clear the 3rd bit of `n = 13 (1101)`:
  - `1 << 3` is `1000`.
  - `~(1000)` is `...0111`.
  - `1101 & ...0111 = 0101` (which is 5).

>[!example]- C++
>```cpp
>int clearBit(int n, int i) {
>    return n & ~(1 << i);
>}
>
>// Example usage
>int n = 13;  // binary: 1101
>int result = clearBit(n, 3);  // result = 5, binary: 0101
>```

>[!example]- Java
>```java
>public int clearBit(int n, int i) {
>    return n & ~(1 << i);
>}
>
>// Example usage
>int n = 13;  // binary: 1101
>int result = clearBit(n, 3);  // result = 5, binary: 0101
>```

>[!example]- Python
>```python
>def clear_bit(n, i):
>    return n & ~(1 << i)
>
># Example usage
>n = 13  # binary: 1101
>result = clear_bit(n, 3)  # result = 5, binary: 0101
>```

>[!example]- JavaScript
>```javascript
>function clearBit(n, i) {
>    return n & ~(1 << i);
>}
>
>// Example usage
>const n = 13;  // binary: 1101
>const result = clearBit(n, 3);  // result = 5, binary: 0101
>```

### 4. Flip the i-th bit
To flip the `i`-th bit (1 to 0, or 0 to 1), you can `XOR` it with a mask that has only the `i`-th bit set.

- **Trick**: `n = n ^ (1 << i)`
- **Example**: To flip the 1st bit of `n = 10 (1010)`:
  - `1 << 1` creates the mask `0010`.
  - `1010 ^ 0010 = 1000` (which is 8).

>[!example]- C++
>```cpp
>int flipBit(int n, int i) {
>    return n ^ (1 << i);
>}
>
>// Example usage
>int n = 10;  // binary: 1010
>int result = flipBit(n, 1);  // result = 8, binary: 1000
>```

>[!example]- Java
>```java
>public int flipBit(int n, int i) {
>    return n ^ (1 << i);
>}
>
>// Example usage
>int n = 10;  // binary: 1010
>int result = flipBit(n, 1);  // result = 8, binary: 1000
>```

>[!example]- Python
>```python
>def flip_bit(n, i):
>    return n ^ (1 << i)
>
># Example usage
>n = 10  # binary: 1010
>result = flip_bit(n, 1)  # result = 8, binary: 1000
>```

>[!example]- JavaScript
>```javascript
>function flipBit(n, i) {
>    return n ^ (1 << i);
>}
>
>// Example usage
>const n = 10;  // binary: 1010
>const result = flipBit(n, 1);  // result = 8, binary: 1000
>```

### 5. Isolate the Last Set (Lowest) Bit
This trick isolates the rightmost '1' bit, setting all other bits to '0'. This is very useful in Fenwick Trees and other algorithms.

- **Trick**: `last_set_bit = n & -n`
- **How it works**: In two's complement representation, `-n` is equivalent to `~n + 1`. This operation effectively flips all the bits up to and including the rightmost '1', and then adding 1 flips them back, leaving only the rightmost '1' set.
- **Example**: For `n = 12 (1100)`:
  - `-n` is `0100`.
  - `1100 & 0100 = 0100` (which is 4).

>[!example]- C++
>```cpp
>int isolateLastSetBit(int n) {
>    return n & -n;
>}
>
>// Example usage
>int n = 12;  // binary: 1100
>int lastBit = isolateLastSetBit(n);  // lastBit = 4, binary: 0100
>
>// Another example
>int n2 = 18;  // binary: 10010
>int lastBit2 = isolateLastSetBit(n2);  // lastBit2 = 2, binary: 00010
>```

>[!example]- Java
>```java
>public int isolateLastSetBit(int n) {
>    return n & -n;
>}
>
>// Example usage
>int n = 12;  // binary: 1100
>int lastBit = isolateLastSetBit(n);  // lastBit = 4, binary: 0100
>
>// Another example
>int n2 = 18;  // binary: 10010
>int lastBit2 = isolateLastSetBit(n2);  // lastBit2 = 2, binary: 00010
>```

>[!example]- Python
>```python
>def isolate_last_set_bit(n):
>    return n & -n
>
># Example usage
>n = 12  # binary: 1100
>last_bit = isolate_last_set_bit(n)  # last_bit = 4, binary: 0100
>
># Another example
>n2 = 18  # binary: 10010
>last_bit2 = isolate_last_set_bit(n2)  # last_bit2 = 2, binary: 00010
>```

>[!example]- JavaScript
>```javascript
>function isolateLastSetBit(n) {
>    return n & -n;
>}
>
>// Example usage
>const n = 12;  // binary: 1100
>const lastBit = isolateLastSetBit(n);  // lastBit = 4, binary: 0100
>
>// Another example
>const n2 = 18;  // binary: 10010
>const lastBit2 = isolateLastSetBit(n2);  // lastBit2 = 2, binary: 00010
>```

### 6. Clear the Last Set (Lowest) Bit
This trick is famously used to count the number of set bits in a number.

- **Trick**: `n = n & (n - 1)`
- **How it works**: Subtracting 1 from a number flips the rightmost '1' to a '0' and all the '0's to its right to '1's. `AND`ing this with the original number cancels out the rightmost '1'.
- **Example**: For `n = 12 (1100)`:
  - `n - 1` is `11 (1011)`.
  - `1100 & 1011 = 1000` (which is 8). The last set bit has been cleared.

>[!example]- C++
>```cpp
>int clearLastSetBit(int n) {
>    return n & (n - 1);
>}
>
>// Count number of set bits using this trick
>int countSetBits(int n) {
>    int count = 0;
>    while (n > 0) {
>        n = n & (n - 1);  // Clear the last set bit
>        count++;
>    }
>    return count;
>}
>
>// Example usage
>int n = 12;  // binary: 1100
>int result = clearLastSetBit(n);  // result = 8, binary: 1000
>int bits = countSetBits(12);  // bits = 2
>```

>[!example]- Java
>```java
>public int clearLastSetBit(int n) {
>    return n & (n - 1);
>}
>
>// Count number of set bits using this trick
>public int countSetBits(int n) {
>    int count = 0;
>    while (n > 0) {
>        n = n & (n - 1);  // Clear the last set bit
>        count++;
>    }
>    return count;
>}
>
>// Example usage
>int n = 12;  // binary: 1100
>int result = clearLastSetBit(n);  // result = 8, binary: 1000
>int bits = countSetBits(12);  // bits = 2
>```

>[!example]- Python
>```python
>def clear_last_set_bit(n):
>    return n & (n - 1)
>
># Count number of set bits using this trick
>def count_set_bits(n):
>    count = 0
>    while n > 0:
>        n = n & (n - 1)  # Clear the last set bit
>        count += 1
>    return count
>
># Example usage
>n = 12  # binary: 1100
>result = clear_last_set_bit(n)  # result = 8, binary: 1000
>bits = count_set_bits(12)  # bits = 2
>```

>[!example]- JavaScript
>```javascript
>function clearLastSetBit(n) {
>    return n & (n - 1);
>}
>
>// Count number of set bits using this trick
>function countSetBits(n) {
>    let count = 0;
>    while (n > 0) {
>        n = n & (n - 1);  // Clear the last set bit
>        count++;
>    }
>    return count;
>}
>
>// Example usage
>const n = 12;  // binary: 1100
>const result = clearLastSetBit(n);  // result = 8, binary: 1000
>const bits = countSetBits(12);  // bits = 2
>```

### 7. Check if a number is a power of two
A number is a power of two if and only if it has exactly one bit set to '1' in its binary representation (e.g., 1, 2, 4, 8...).

- **Trick**: `return n > 0 and (n & (n - 1)) == 0`
- **How it works**: If `n` is a power of two, it has only one '1' bit. `n - 1` will have all bits to the right of that '1' set to '1'. `AND`ing them together will always result in 0. The `n > 0` check handles the edge case of `n=0`.
- **Example**: For `n = 8 (1000)`:
  - `n - 1` is `7 (0111)`.
  - `1000 & 0111 = 0`. It's a power of two.
- **Example**: For `n = 12 (1100)`:
  - `n - 1` is `11 (1011)`.
  - `1100 & 1011 = 1000` (non-zero). Not a power of two.

>[!example]- C++
>```cpp
>bool isPowerOfTwo(int n) {
>    return n > 0 && (n & (n - 1)) == 0;
>}
>
>// Example usage
>bool result1 = isPowerOfTwo(8);   // true
>bool result2 = isPowerOfTwo(12);  // false
>bool result3 = isPowerOfTwo(16);  // true
>bool result4 = isPowerOfTwo(0);   // false
>```

>[!example]- Java
>```java
>public boolean isPowerOfTwo(int n) {
>    return n > 0 && (n & (n - 1)) == 0;
>}
>
>// Example usage
>boolean result1 = isPowerOfTwo(8);   // true
>boolean result2 = isPowerOfTwo(12);  // false
>boolean result3 = isPowerOfTwo(16);  // true
>boolean result4 = isPowerOfTwo(0);   // false
>```

>[!example]- Python
>```python
>def is_power_of_two(n):
>    return n > 0 and (n & (n - 1)) == 0
>
># Example usage
>result1 = is_power_of_two(8)   # True
>result2 = is_power_of_two(12)  # False
>result3 = is_power_of_two(16)  # True
>result4 = is_power_of_two(0)   # False
>```

>[!example]- JavaScript
>```javascript
>function isPowerOfTwo(n) {
>    return n > 0 && (n & (n - 1)) === 0;
>}
>
>// Example usage
>const result1 = isPowerOfTwo(8);   // true
>const result2 = isPowerOfTwo(12);  // false
>const result3 = isPowerOfTwo(16);  // true
>const result4 = isPowerOfTwo(0);   // false
>```
