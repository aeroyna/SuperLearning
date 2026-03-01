# Sealed Classes: Architectural Control

Sealed classes enable "Algebraic Data Types" (ADTs) in Java.

## 1. The Problem with `final`
*   `final class`: Nobody can extend it. (Too restrictive).
*   `public class`: Anybody can extend it. (Too open).

## 2. Sealed Solution
`sealed` allows you to say: "This class can be extended, but *only* by these specific classes."

```java
public sealed interface Shape permits Circle, Square { }
```

## 3. Exhaustiveness in Switch
When used with Pattern Matching, the compiler knows the hierarchy is closed.

```java
Shape s = ...;
// Compile-time check: Have I covered all shapes?
// No 'default' needed!
switch (s) {
    case Circle c -> ...
    case Square q -> ...
}
```
If you add `Triangle` to `Shape` later, this switch statement becomes a **compile error**, alerting you to handle the new case. This makes maintenance incredibly safe.
