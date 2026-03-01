# Pattern Matching (Java 16+)

Pattern matching simplifies the testing of an object's type and the extraction of its components.

## 1. `instanceof`
Before:
```java
if (obj instanceof String) {
    String s = (String) obj; // Boring cast
    System.out.println(s.length());
}
```

After (Java 16):
```java
if (obj instanceof String s) {
    System.out.println(s.length()); // 's' is already cast
}
```

## 2. Record Patterns (Java 21)
Deconstructing records directly.

```java
record Point(int x, int y) {}

if (obj instanceof Point(int x, int y)) {
    System.out.println("x: " + x + ", y: " + y);
}
```