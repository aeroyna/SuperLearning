# Bitwise Operators

Bitwise operators perform operations on individual bits of a number.

| Operator | Description      | Example     |
|----------|------------------|-------------|
| `&`      | Bitwise AND      | `a & b`     |
| `|`      | Bitwise OR       | `a | b`     |
| `^`      | Bitwise XOR      | `a ^ b`     |
| `~`      | Bitwise NOT      | `~a`        |
| `<<`     | Left Shift       | `a << 2`    |
| `>>`     | Right Shift      | `a >> 2`    |

## Truth Tables

Let's consider two bits, `x` and `y`.

### Bitwise AND (`&`)

| x | y | x & y |
|---|---|-------|
| 0 | 0 | 0     |
| 0 | 1 | 0     |
| 1 | 0 | 0     |
| 1 | 1 | 1     |

### Bitwise OR (`|`)

| x | y | x \| y |
|---|---|--------|
| 0 | 0 | 0      |
| 0 | 1 | 1      |
| 1 | 0 | 1      |
| 1 | 1 | 1      |

### Bitwise XOR (`^`)

| x | y | x ^ y |
|---|---|-------|
| 0 | 0 | 0     |
| 0 | 1 | 1     |
| 1 | 0 | 1     |
| 1 | 1 | 0     |

### Example

Let's take `a = 5` (binary `0101`) and `b = 3` (binary `0011`).

*   `a & b`:
    ```
      0101
    & 0011
    ------
      0001  (decimal 1)
    ```

*   `a | b`:
    ```
      0101
    | 0011
    ------
      0111  (decimal 7)
    ```

*   `a ^ b`:
    ```
      0101
    ^ 0011
    ------
      0110  (decimal 6)
    ```

*   `~a`:
    ```
    ~ 00000000 00000000 00000000 00000101
    -----------------------------------
      11111111 11111111 11111111 11111010  (decimal -6 in two's complement)
    ```

## Shift Operators

*   **Left Shift (`<<`):** Shifts the bits to the left and fills the empty spaces with zeros. `a << n` is equivalent to multiplying `a` by 2<sup>n</sup>.

    `5 << 2`:
    `0101` becomes `10100` (decimal 20)

*   **Right Shift (`>>`):** Shifts the bits to the right. The behavior of filling the empty spaces on the left depends on the type of the variable (signed or unsigned). For unsigned numbers, it fills with zeros. For signed numbers, it might fill with the sign bit (arithmetic shift) or zeros (logical shift), which is implementation-defined. `a >> n` is equivalent to dividing `a` by 2<sup>n</sup>.

    `5 >> 2`:
    `0101` becomes `0001` (decimal 1)

### Code Example

```cpp
#include <iostream>

int main() {
    int a = 5;  // 0101
    int b = 3;  // 0011

    std::cout << "a & b = " << (a & b) << std::endl;
    std::cout << "a | b = " << (a | b) << std::endl;
    std::cout << "a ^ b = " << (a ^ b) << std::endl;
    std::cout << "~a = " << (~a) << std::endl;
    std::cout << "a << 2 = " << (a << 2) << std::endl;
    std::cout << "a >> 2 = " << (a >> 2) << std::endl;

    return 0;
}
```
