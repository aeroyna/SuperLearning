# Operators and Expressions

Java provides a rich set of operators to manipulate variables.

## 1. Arithmetic Operators
Basic math operations.
*   `+` (Addition)
*   `-` (Subtraction)
*   `*` (Multiplication)
*   `/` (Division)
    *   **Integer Division:** `5 / 2` equals `2` (truncates decimal).
    *   **Float Division:** `5.0 / 2` equals `2.5`.
*   `%` (Modulus/Remainder)
    *   `5 % 2` equals `1`.
*   `++` (Increment) / `--` (Decrement)
    *   **Pre (`++i`):** Increment, then use value.
    *   **Post (`i++`):** Use value, then increment.

## 2. Relational Operators
Used for comparison. Returns `boolean` (`true` or `false`).
*   `==` (Equal to)
*   `!=` (Not equal to)
*   `>` (Greater than)
*   `<` (Less than)
*   `>=` (Greater than or equal)
*   `<=` (Less than or equal)

## 3. Logical Operators
Used to combine boolean expressions.
*   `&&` (Logical AND): True if **both** are true.
*   `||` (Logical OR): True if **at least one** is true.
*   `!` (Logical NOT): Inverts value (`!true` is `false`).

### Short-Circuit Evaluation
*   `&&`: If the left side is `false`, the right side is **ignored** (skipped).
    ```java
    if (obj != null && obj.isValid()) { ... } // Safe: isValid() won't run if obj is null
    ```
*   `||`: If the left side is `true`, the right side is ignored.

## 4. Bitwise Operators
Operations on individual bits (0s and 1s). Fast and low-level.
*   `&` (Bitwise AND)
*   `|` (Bitwise OR)
*   `^` (Bitwise XOR): 1 if bits are different.
*   `~` (Bitwise Complement): Inverts all bits.
*   `<<` (Left Shift): Shifts bits left, fills with 0. Effectively multiplies by 2^n.
*   `>>` (Signed Right Shift): Shifts bits right, preserves sign bit.
*   `>>>` (Unsigned Right Shift): Shifts bits right, fills with 0 (always positive result).

## 5. Assignment Operators
*   `=` (Simple assignment)
*   `+=`, `-=`, `*=`, `/=`, `%=` (Compound assignment)
    *   `a += 5` is roughly `a = a + 5`.
    *   **Implicit Casting:** Compound operators automatically cast the result.
    ```java
    byte b = 1;
    // b = b + 1; // Error: result is int
    b += 1;       // OK: internally (byte)(b + 1)
    ```

## 6. Ternary Operator (Conditional)
A one-line `if-else`.
```java
// type variable = (condition) ? valueIfTrue : valueIfFalse;
int max = (a > b) ? a : b;
```

## 7. Operator Precedence
Order of operations (Highest to Lowest):
1.  Postfix: `expr++`, `expr--`
2.  Unary: `++expr`, `--expr`, `+`, `-`, `~`, `!`
3.  Multiplicative: `*`, `/`, `%`
4.  Additive: `+`, `-`
5.  Shift: `<<`, `>>`, `>>>`
6.  Relational: `<`, `>`, `<=`, `>=`
7.  Equality: `==`, `!=`
8.  Bitwise AND: `&`
9.  Bitwise XOR: `^`
10. Bitwise OR: `|`
11. Logical AND: `&&`
12. Logical OR: `||`
13. Ternary: `? :`
14. Assignment: `=`, `+=`, etc.