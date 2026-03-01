# Records: Data-Oriented Programming

Records are more than just concise syntax. They are "transparent carriers for immutable data."

## 1. Semantics
A Record creates a strong contract: "The state of this object is *exactly* defined by its constructor components."
*   **Immutable:** Fields are `private final`. No setters.
*   **Final:** The record class itself is `final`.

## 2. Compact Constructors
Records allow a unique "Compact Constructor" syntax for validation without re-listing fields.

```java
public record Range(int start, int end) {
    // No parameter list!
    public Range {
        if (end < start) {
            throw new IllegalArgumentException("End < Start");
        }
        // Implicitly assigns: this.start = start, this.end = end
    }
}
```

## 3. Serialization Superpower
Records are serialized differently than classes.
*   **Classes:** Uses magic (Unsafe) to bypass the constructor and set fields directly. Vulnerable to invalid states.
*   **Records:** **Always calls the canonical constructor** during deserialization.
    *   *Result:* Impossible to create a Record with an invalid state (e.g., negative range) via deserialization attacks. This is a huge security win.
