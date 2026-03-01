# Operators and Expressions: The Language of Computation

Operators are special symbols that perform operations on one or more operands. Expressions are combinations of variables, literals, and operators that evaluate to a single value. Mastering operators is fundamental to writing any computational logic in Java.

## 1. Arithmetic Operators

Used for basic mathematical calculations.

*   `+` (Addition): `5 + 3` results in `8`. Also used for String concatenation (`"Hello" + "World"`).
*   `-` (Subtraction): `10 - 4` results in `6`.
*   `*` (Multiplication): `3 * 7` results in `21`.
*   `/` (Division):
    *   **Integer Division:** When both operands are integers, the result is an integer, and any fractional part is **truncated** (discarded). `5 / 2` results in `2`. `int result = 7 / 3; // result is 2`.
    *   **Floating-Point Division:** If at least one operand is a floating-point type (`float` or `double`), the result is a floating-point type. `5.0 / 2` results in `2.5`.
    *   **Division by Zero:** Integer division by zero (`10 / 0`) throws `ArithmeticException`. Floating-point division by zero (`10.0 / 0.0`) results in `Infinity` or `NaN` (Not a Number) and does *not* throw an exception.
*   `%` (Modulus/Remainder): Returns the remainder of a division. `5 % 2` results in `1`. `10 % 3` results in `1`. The sign of the result matches the sign of the *dividend*. `-5 % 2` results in `-1`.

### Unary Operators
*   `++` (Increment): Adds 1 to the operand.
*   `--` (Decrement): Subtracts 1 from the operand.
    *   **Pre-increment/decrement (`++i`, `--i`):** The operation is performed *before* the value is used in the expression.
        ```java
        int i = 5;
        int j = ++i; // i becomes 6, j becomes 6
        ```
    *   **Post-increment/decrement (`i++`, `i--`):** The value is used in the expression *before* the operation is performed.
        ```java
        int i = 5;
        int j = i++; // j becomes 5, then i becomes 6
        ```

## 2. Relational Operators

Used to compare two operands. They always return a `boolean` value (`true` or `false`).

*   `==` (Equal to): Returns `true` if both operands have the same value. For objects, it checks if they refer to the **same memory location**. (Content comparison for strings needs `.equals()`).
*   `!=` (Not equal to): Returns `true` if operands have different values.
*   `>` (Greater than): `5 > 3` is `true`.
*   `<` (Less than): `5 < 3` is `false`.
*   `>=` (Greater than or equal to).
*   `<=` (Less than or equal to).

## 3. Logical Operators

Used to combine boolean expressions.

*   `&&` (Logical AND): Returns `true` if **both** operands are `true`.
    *   **Short-Circuiting:** If the left operand is `false`, the right operand is **not evaluated**. This is crucial for avoiding `NullPointerException`.
        ```java
        if (obj != null && obj.isValid()) { // obj.isValid() is only called if obj is not null
            // ...
        }
        ```
*   `||` (Logical OR): Returns `true` if **at least one** operand is `true`.
    *   **Short-Circuiting:** If the left operand is `true`, the right operand is **not evaluated**.
        ```java
        if (hasPermission() || isAuthenticated()) { // isAuthenticated() is only called if hasPermission() is false
            // ...
        }
        ```
*   `!` (Logical NOT): Inverts the boolean value of its operand. `!true` is `false`.
*   `&` (Bitwise AND / Non-short-circuiting AND): Evaluates both operands even if the left is `false`. Rarely used for boolean logic.
*   `|` (Bitwise OR / Non-short-circuiting OR): Evaluates both operands even if the left is `true`. Rarely used for boolean logic.

## 4. Bitwise Operators

Operate on individual bits of integer types (`byte`, `short`, `int`, `long`).

*   `&` (Bitwise AND): Sets each bit to 1 if both corresponding bits are 1.
*   `|` (Bitwise OR): Sets each bit to 1 if one or both corresponding bits are 1.
*   `^` (Bitwise XOR): Sets each bit to 1 if corresponding bits are different.
*   `~` (Bitwise Complement): Inverts all bits (flips 0s to 1s and 1s to 0s). `~0b00000001` results in `0b11111110` (for `byte`).
*   `<<` (Left Shift): Shifts bits to the left, filling new bits with zeros. Effectively multiplies by powers of 2. `5 << 1` (0101 << 1) results in `10` (1010).
*   `>>` (Signed Right Shift): Shifts bits to the right, filling new bits on the left with the value of the most significant bit (preserves sign). Effectively divides by powers of 2. `-8 >> 1` results in `-4`.
*   `>>>` (Unsigned Right Shift): Shifts bits to the right, filling new bits on the left with zeros. Always results in a positive number (unless the original number was `0`). `-8 >>> 1` results in `2147483644` (for `int`).

## 5. Assignment Operators

Used to assign values to variables.

*   `=` (Simple assignment): `x = 10;`
*   Compound Assignment Operators: Combine an arithmetic/bitwise operator with assignment.
    *   `+=` (Add and assign): `x += 5;` is equivalent to `x = x + 5;`
    *   `-=` (Subtract and assign)
    *   `*=` (Multiply and assign)
    *   `/=` (Divide and assign)
    *   `%=` (Modulus and assign)
    *   `&=` (Bitwise AND and assign)
    *   `|=` (Bitwise OR and assign)
    *   `^=` (Bitwise XOR and assign)
    *   `<<=` (Left shift and assign)
    *   `>>=` (Signed right shift and assign)
    *   `>>>=` (Unsigned right shift and assign)

### Nuance: Compound Assignment and Type Casting
Compound assignment operators perform an implicit narrowing cast if necessary. This can be convenient but also hides potential data loss.

```java
byte b = 10;
b += 5; // Valid: Internally Java does b = (byte)(b + 5);
// byte b = 10; b = b + 5; // Compile-time error: Possible lossy conversion from int to byte
```

## 6. Operator Precedence and Associativity

**Precedence** determines the order in which operators are evaluated (e.g., `*` before `+`).
**Associativity** determines the order in which operators of the same precedence are evaluated (e.g., left-to-right or right-to-left).

### Highest Precedence to Lowest (Common Operators)
1.  **Postfix:** `expr++`, `expr--`
2.  **Unary:** `++expr`, `--expr`, `+expr`, `-expr`, `~`, `!`
3.  **Multiplicative:** `*`, `/`, `%` (Left-to-right)
4.  **Additive:** `+`, `-` (Left-to-right)
5.  **Shift:** `<<`, `>>`, `>>>` (Left-to-right)
6.  **Relational:** `<`, `>`, `<=`, `>=` (Left-to-right)
7.  **Equality:** `==`, `!=` (Left-to-right)
8.  **Bitwise AND:** `&` (Left-to-right)
9.  **Bitwise XOR:** `^` (Left-to-right)
10. **Bitwise OR:** `|` (Left-to-right)
11. **Logical AND:** `&&` (Left-to-right)
12. **Logical OR:** `||` (Left-to-right)
13. **Ternary (Conditional):** `? :` (Right-to-left)
14. **Assignment:** `=`, `+=`, `-=`, etc. (Right-to-left)

*   **Best Practice:** When in doubt, use parentheses `()` to explicitly define the order of evaluation and improve code readability, even if current precedence rules would yield the same result. This makes the code less prone to errors during future modifications.

Mastering Java's operators and their subtleties is essential for writing accurate, efficient, and bug-free code.

---

### Links to Topics:
*   [Variables & Primitive Types](01_variables_and_primitive_types.md)
*   [Operators & Expressions](02_operators_and_expressions.md)
*   [Arrays](03_arrays.md)
*   [Type Casting & Conversion](04_type_casting_and_conversion.md)
