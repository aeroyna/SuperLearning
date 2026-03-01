# Type Casting and Conversion

Converting a value from one data type to another is common in programming.

## 1. Widening Casting (Implicit)
Converting a **smaller** type to a **larger** type size.
*   **Safe:** No data is lost.
*   **Automatic:** Java handles this.

**Order:**
`byte` -> `short` -> `char` -> `int` -> `long` -> `float` -> `double`

```java
int myInt = 9;
double myDouble = myInt; // Automatic casting: 9.0
```

## 2. Narrowing Casting (Explicit)
Converting a **larger** type to a **smaller** type.
*   **Unsafe:** Potential data loss or integer overflow.
*   **Manual:** You must specify the type in parentheses `()`.

```java
double myDouble = 9.78d;
int myInt = (int) myDouble; // Manual casting: 9 (truncates decimal)
```

### Overflow Example
```java
int val = 130;
byte b = (byte) val; 
// Result: -126
// Why? 130 is 10000010 in binary.
// Byte is signed 8-bit. The leading 1 makes it negative in Two's Complement.
```

## 3. Type Promotion in Expressions
When evaluating an expression (e.g., `a + b`), Java promotes operands to larger types automatically.

### Rules
1.  If either operand is `double`, the other is converted to `double`.
2.  Else if either is `float`, the other is converted to `float`.
3.  Else if either is `long`, the other is converted to `long`.
4.  **Important:** Otherwise, both are converted to `int` (even if they are `byte` or `short`).

```java
byte b1 = 10;
byte b2 = 20;
// byte sum = b1 + b2; // ERROR: b1 + b2 results in 'int'
byte sum = (byte) (b1 + b2); // Fixed
```

## 4. String Conversion
### Primitive to String
```java
String s1 = String.valueOf(100);
String s2 = Integer.toString(100);
String s3 = "" + 100; // Concise but slightly less efficient
```

### String to Primitive
Each wrapper class provides parsing methods.
```java
int i = Integer.parseInt("123");
double d = Double.parseDouble("3.14");
boolean b = Boolean.parseBoolean("true");
```
**Exception:** `NumberFormatException` throws if the string is invalid (e.g., "abc").