## Core Bitwise Operations

Bit manipulation is performed using a set of low-level operators that act directly on the binary representations of integers. Mastering these operators is the first step to solving bitwise problems.

Let's consider two example 8-bit numbers:
`a = 93`  (binary `01011101`)
`b = 55`  (binary `00110111`)

---

### 1. AND (`&`)
The `AND` operator compares two bits and returns `1` only if **both** bits are `1`. It is commonly used to "check" or "clear" a specific bit.

- **Check a bit**: To see if the `k`-th bit is set, you can `AND` the number with a mask that has only the `k`-th bit set (i.e., `1 << k`). If the result is non-zero, the bit was set.
- **Example**: `a & b`
  ```
    01011101  (93)
  & 00110111  (55)
    --------
    00010101  (21)
  ```

>[!example]- C++
>```cpp
>int bitwiseAnd(int a, int b) {
>    return a & b;
>}
>
>// Example usage
>int a = 93;
>int b = 55;
>int result = bitwiseAnd(a, b);  // result = 21
>```

>[!example]- Java
>```java
>public int bitwiseAnd(int a, int b) {
>    return a & b;
>}
>
>// Example usage
>int a = 93;
>int b = 55;
>int result = bitwiseAnd(a, b);  // result = 21
>```

>[!example]- Python
>```python
>def bitwise_and(a, b):
>    return a & b
>
># Example usage
>a = 93
>b = 55
>result = bitwise_and(a, b)  # result = 21
>```

>[!example]- JavaScript
>```javascript
>function bitwiseAnd(a, b) {
>    return a & b;
>}
>
>// Example usage
>const a = 93;
>const b = 55;
>const result = bitwiseAnd(a, b);  // result = 21
>```

---

### 2. OR (`|`)
The `OR` operator compares two bits and returns `1` if **at least one** of the bits is `1`. It is commonly used to "set" a specific bit to 1.

- **Set a bit**: To set the `k`-th bit, you can `OR` the number with a mask `1 << k`.
- **Example**: `a | b`
  ```
    01011101  (93)
  | 00110111  (55)
    --------
    01111111  (127)
  ```

>[!example]- C++
>```cpp
>int bitwiseOr(int a, int b) {
>    return a | b;
>}
>
>// Example usage
>int a = 93;
>int b = 55;
>int result = bitwiseOr(a, b);  // result = 127
>```

>[!example]- Java
>```java
>public int bitwiseOr(int a, int b) {
>    return a | b;
>}
>
>// Example usage
>int a = 93;
>int b = 55;
>int result = bitwiseOr(a, b);  // result = 127
>```

>[!example]- Python
>```python
>def bitwise_or(a, b):
>    return a | b
>
># Example usage
>a = 93
>b = 55
>result = bitwise_or(a, b)  # result = 127
>```

>[!example]- JavaScript
>```javascript
>function bitwiseOr(a, b) {
>    return a | b;
>}
>
>// Example usage
>const a = 93;
>const b = 55;
>const result = bitwiseOr(a, b);  // result = 127
>```

---

### 3. XOR (`^`)
The `XOR` (exclusive OR) operator compares two bits and returns `1` only if the bits are **different**. It is a versatile operator with several unique properties.

- **Properties**:
  - `x ^ x = 0` (XORing a number with itself results in zero).
  - `x ^ 0 = x` (XORing with zero does not change the number).
  - `x ^ y = y ^ x` (Commutative).
  - `(x ^ y) ^ z = x ^ (y ^ z)` (Associative).
- **Use Cases**:
  - **Flipping a bit**: `number ^ (1 << k)` will flip the `k`-th bit.
  - **Finding the unique number**: If you have a list of numbers where every number appears twice except for one, XORing all the numbers together will cancel out all the pairs, leaving only the unique number.
- **Example**: `a ^ b`
  ```
    01011101  (93)
  ^ 00110111  (55)
    --------
    01101010  (106)
  ```

>[!example]- C++
>```cpp
>int bitwiseXor(int a, int b) {
>    return a ^ b;
>}
>
>// Finding unique number in array where all others appear twice
>int findUnique(vector<int>& nums) {
>    int result = 0;
>    for (int num : nums) {
>        result ^= num;
>    }
>    return result;
>}
>
>// Example usage
>int a = 93;
>int b = 55;
>int result = bitwiseXor(a, b);  // result = 106
>```

>[!example]- Java
>```java
>public int bitwiseXor(int a, int b) {
>    return a ^ b;
>}
>
>// Finding unique number in array where all others appear twice
>public int findUnique(int[] nums) {
>    int result = 0;
>    for (int num : nums) {
>        result ^= num;
>    }
>    return result;
>}
>
>// Example usage
>int a = 93;
>int b = 55;
>int result = bitwiseXor(a, b);  // result = 106
>```

>[!example]- Python
>```python
>def bitwise_xor(a, b):
>    return a ^ b
>
># Finding unique number in array where all others appear twice
>def find_unique(nums):
>    result = 0
>    for num in nums:
>        result ^= num
>    return result
>
># Example usage
>a = 93
>b = 55
>result = bitwise_xor(a, b)  # result = 106
>```

>[!example]- JavaScript
>```javascript
>function bitwiseXor(a, b) {
>    return a ^ b;
>}
>
>// Finding unique number in array where all others appear twice
>function findUnique(nums) {
>    let result = 0;
>    for (const num of nums) {
>        result ^= num;
>    }
>    return result;
>}
>
>// Example usage
>const a = 93;
>const b = 55;
>const result = bitwiseXor(a, b);  // result = 106
>```

---

### 4. NOT (`~`)
The `NOT` operator, or bitwise complement, inverts all the bits of a number. `0`s become `1`s and `1`s become `0`s. In most languages, this is based on the two's complement representation of signed integers, so `~x` is equivalent to `-x - 1`.

- **Example**: `~a`
  ```
  ~ 01011101  (93)
    --------
    10100010  (-94 in two's complement)
  ```

>[!example]- C++
>```cpp
>int bitwiseNot(int a) {
>    return ~a;
>}
>
>// Example usage
>int a = 93;
>int result = bitwiseNot(a);  // result = -94
>```

>[!example]- Java
>```java
>public int bitwiseNot(int a) {
>    return ~a;
>}
>
>// Example usage
>int a = 93;
>int result = bitwiseNot(a);  // result = -94
>```

>[!example]- Python
>```python
>def bitwise_not(a):
>    return ~a
>
># Example usage
>a = 93
>result = bitwise_not(a)  # result = -94
>```

>[!example]- JavaScript
>```javascript
>function bitwiseNot(a) {
>    return ~a;
>}
>
>// Example usage
>const a = 93;
>const result = bitwiseNot(a);  // result = -94
>```

---

### 5. Bit Shifts (`<<` and `>>`)
These operators shift all bits of a number to the left or right.

- **Left Shift (`<<`)**: `x << k` shifts the bits of `x` to the left by `k` positions, filling the empty spots on the right with zeros. This is equivalent to multiplying `x` by `2^k`.
  - **Example**: `93 << 1` (`01011101` becomes `10111010`, which is 186).

- **Right Shift (`>>`)**: `x >> k` shifts the bits of `x` to the right by `k` positions. For positive numbers, the empty spots on the left are filled with zeros. This is equivalent to integer division of `x` by `2^k`.
  - **Example**: `93 >> 1` (`01011101` becomes `00101110`, which is 46).

>[!example]- C++
>```cpp
>int leftShift(int x, int k) {
>    return x << k;
>}
>
>int rightShift(int x, int k) {
>    return x >> k;
>}
>
>// Example usage
>int x = 93;
>int left = leftShift(x, 1);    // left = 186
>int right = rightShift(x, 1);  // right = 46
>
>// Multiply/divide by powers of 2
>int multiplyBy8 = x << 3;      // x * 8 = 744
>int divideBy4 = x >> 2;        // x / 4 = 23
>```

>[!example]- Java
>```java
>public int leftShift(int x, int k) {
>    return x << k;
>}
>
>public int rightShift(int x, int k) {
>    return x >> k;
>}
>
>// Example usage
>int x = 93;
>int left = leftShift(x, 1);    // left = 186
>int right = rightShift(x, 1);  // right = 46
>
>// Multiply/divide by powers of 2
>int multiplyBy8 = x << 3;      // x * 8 = 744
>int divideBy4 = x >> 2;        // x / 4 = 23
>```

>[!example]- Python
>```python
>def left_shift(x, k):
>    return x << k
>
>def right_shift(x, k):
>    return x >> k
>
># Example usage
>x = 93
>left = left_shift(x, 1)    # left = 186
>right = right_shift(x, 1)  # right = 46
>
># Multiply/divide by powers of 2
>multiply_by_8 = x << 3     # x * 8 = 744
>divide_by_4 = x >> 2       # x / 4 = 23
>```

>[!example]- JavaScript
>```javascript
>function leftShift(x, k) {
>    return x << k;
>}
>
>function rightShift(x, k) {
>    return x >> k;
>}
>
>// Example usage
>const x = 93;
>const left = leftShift(x, 1);    // left = 186
>const right = rightShift(x, 1);  // right = 46
>
>// Multiply/divide by powers of 2
>const multiplyBy8 = x << 3;      // x * 8 = 744
>const divideBy4 = x >> 2;        // x / 4 = 23
>```
