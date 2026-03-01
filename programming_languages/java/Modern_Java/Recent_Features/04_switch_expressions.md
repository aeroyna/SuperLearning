# Switch Expressions (Java 14)

Switch expressions allow `switch` to be used as an expression (return a value) rather than just a statement.

## 1. Syntax
*   Uses `->` syntax.
*   No fall-through.
*   Must cover all cases (exhaustive).

```java
String dayType = switch (day) {
    case MONDAY, FRIDAY -> "Work";
    case SATURDAY, SUNDAY -> "Rest";
    default -> "Unknown";
};
```

## 2. Pattern Matching for Switch (Java 21)
Switch can now switch on **types**.

```java
String result = switch (obj) {
    case Integer i -> String.format("int %d", i);
    case String s  -> String.format("string %s", s);
    case null      -> "null object";
    default        -> "unknown";
};
```