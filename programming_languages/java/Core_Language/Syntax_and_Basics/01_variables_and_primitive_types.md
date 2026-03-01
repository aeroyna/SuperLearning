# Variables and Primitive Types

## 1. Variables
A variable is a container that holds data. In Java, every variable must have a **Type**.

### Declaration Syntax
```java
// type name = value;
int age = 25;
```

### Naming Conventions (CamelCase)
*   **Variables/Methods:** `studentName`, `totalScore`, `isAvailable`. (Start lowercase).
*   **Classes/Interfaces:** `Student`, `Car`. (Start uppercase).
*   **Constants:** `MAX_PRIORITY`, `PI`. (All uppercase with underscores).

### Rules
*   Case-sensitive (`age` vs `Age`).
*   Can start with letter, `_`, or `$`.
*   Cannot start with a number.
*   Cannot use reserved keywords (`class`, `public`, `void`).

---

## 2. Primitive Data Types
Java has **8 Primitive Types**. These are the building blocks of data. They are stored directly in the **Stack Memory** (for local variables) and hold the actual value, not a reference.

### Integer Types (Signed Two's Complement)
1.  **`byte`**
    *   **Size:** 8-bit
    *   **Range:** -128 to 127
    *   **Use:** Saving memory in large arrays, binary data.
2.  **`short`**
    *   **Size:** 16-bit
    *   **Range:** -32,768 to 32,767
    *   **Use:** Rare. 16-bit processors.
3.  **`int`** (Default for integer numbers)
    *   **Size:** 32-bit
    *   **Range:** -2^31 to 2^31-1 (~2 billion)
    *   **Use:** Default for math and counters.
4.  **`long`**
    *   **Size:** 64-bit
    *   **Range:** -2^63 to 2^63-1 (Huge)
    *   **Suffix:** Must use `L` or `l` (e.g., `9000000000L`).
    *   **Use:** Timestamps, file sizes, large IDs.

### Floating-Point Types (IEEE 754)
1.  **`float`**
    *   **Size:** 32-bit (Single precision)
    *   **Suffix:** Must use `F` or `f` (e.g., `3.14f`).
    *   **Use:** Graphics libraries, low memory. **NOT** for currency (precision errors).
2.  **`double`** (Default for decimals)
    *   **Size:** 64-bit (Double precision)
    *   **Use:** Math calculations.

### Character Type
1.  **`char`**
    *   **Size:** 16-bit (Unicode UTF-16)
    *   **Range:** '\u0000' (0) to '\uffff' (65,535)
    *   **Use:** Single character. e.g., `'A'`, `'@'`, `'\u0041'`.
    *   **Note:** Enclosed in **Single Quotes** `' '`.

### Boolean Type
1.  **`boolean`**
    *   **Size:** Virtual Machine dependent (usually 1 bit logically, but 1 byte in arrays).
    *   **Values:** `true` or `false`.
    *   **Use:** Logic control.

| Type | Default Value (Fields) | Size |
| :--- | :--- | :--- |
| boolean | false | 1 bit* |
| byte | 0 | 1 byte |
| short | 0 | 2 bytes |
| char | '\u0000' | 2 bytes |
| int | 0 | 4 bytes |
| float | 0.0f | 4 bytes |
| long | 0L | 8 bytes |
| double | 0.0d | 8 bytes |

> **Note:** Local variables (inside methods) do **not** get default values. You must initialize them before use, or the compiler will throw an error.

---

## 3. Literals and Underscores
Java 7 introduced using underscores for readability in numbers.

```java
int billion = 1_000_000_000; // Same as 1000000000
long creditCard = 1234_5678_9012_3456L;
double pi = 3.141_592;
```
*   Underscores are ignored by the compiler.
*   Cannot put underscore at the start, end, or next to a decimal point.

## 4. `var` Keyword (Type Inference)
Introduced in Java 10. The compiler infers the type based on the value.
```java
var name = "John"; // inferred as String
var age = 25;      // inferred as int
```
*   **Rules:**
    *   Must be initialized immediately.
    *   Cannot be used for method parameters or class fields.
    *   Cannot be null (`var x = null` is illegal).
    *   Does not mean "dynamic typing". Once inferred as `int`, it is always `int`.