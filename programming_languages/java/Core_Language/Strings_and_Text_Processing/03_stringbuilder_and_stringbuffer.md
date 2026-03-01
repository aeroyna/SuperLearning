# StringBuilder and StringBuffer

Due to the immutability of `String` objects, repeated modifications (like concatenation in a loop) can lead to the creation of many intermediate `String` objects, consuming excessive memory and CPU cycles. Java provides `StringBuilder` and `StringBuffer` classes for efficient, mutable string manipulation.

## 1. The Problem with `String` Concatenation

Consider this code:
```java
String result = "";
for (int i = 0; i < 1000; i++) {
    result += i; // result = result + i;
}
```
Each `result += i` operation effectively:
1.  Creates a new `String` object by concatenating `result` and `i`.
2.  Discards the old `result` object (making it eligible for garbage collection).
For 1000 iterations, this creates approximately 1000 `String` objects, which is highly inefficient.

## 2. `StringBuilder` (Mutable and Non-Synchronized)

`StringBuilder` is a mutable sequence of characters. It provides an efficient way to append, insert, delete, or replace characters in a string without creating new `String` objects for every modification.

### Characteristics
*   **Mutable:** Its content can be changed.
*   **Non-Synchronized:** Not thread-safe. If multiple threads access a `StringBuilder` instance concurrently, external synchronization mechanisms are required.
*   **Performance:** Faster than `StringBuffer` in a single-threaded environment due to lack of synchronization overhead.
*   **Introduced:** Java 5.

### Common Methods
*   `append(data)`: Adds data to the end.
*   `insert(offset, data)`: Inserts data at a specified position.
*   `delete(start, end)`: Deletes characters from `start` to `end-1`.
*   `replace(start, end, str)`: Replaces a portion of the string.
*   `reverse()`: Reverses the sequence of characters.
*   `length()`: Returns the number of characters.
*   `capacity()`: Returns the currently allocated capacity.
*   `toString()`: Converts the `StringBuilder` content to an immutable `String` object.

### Example
```java
StringBuilder sb = new StringBuilder(); // Default capacity 16 chars
sb.append("Hello");
sb.append(" ");
sb.append("World");
sb.insert(6, "Beautiful "); // Insert "Beautiful " at index 6

System.out.println(sb); // Output: Hello Beautiful World

sb.delete(5, 15); // Delete " Beautiful" (index 5 to 14)
System.out.println(sb); // Output: HelloWorld

String finalString = sb.toString(); // Convert to immutable String
System.out.println(finalString); // Output: HelloWorld
```

## 3. `StringBuffer` (Mutable and Synchronized)

`StringBuffer` is identical to `StringBuilder` in terms of its API and functionality, but with one critical difference:

### Characteristics
*   **Mutable:** Its content can be changed.
*   **Synchronized:** All its public methods are `synchronized`, making it **thread-safe**. This means multiple threads can safely call methods on a `StringBuffer` instance without corrupting its internal state.
*   **Performance:** Slower than `StringBuilder` in a single-threaded environment due to the overhead of synchronization.
*   **Introduced:** Java 1 (legacy).

### Example
The methods are the same as `StringBuilder`.
```java
StringBuffer sbuf = new StringBuffer();
sbuf.append("Thread");
sbuf.append("Safe");
sbuf.reverse();
System.out.println(sbuf); // Output: efaSderhT
```

## 4. When to Use Which?

| Feature         | `String`                | `StringBuilder`       | `StringBuffer`           |
| :-------------- | :---------------------- | :-------------------- | :----------------------- |
| **Mutability**  | Immutable               | Mutable               | Mutable                  |
| **Thread-Safe** | Yes (inherently)        | No                    | Yes (synchronized)       |
| **Performance** | Poor for modifications  | High                  | Moderate (due to sync)   |
| **Usage**       | Fixed text, keys in maps, multi-threaded read-only access | Single-threaded string manipulation | Multi-threaded string manipulation |

*   **Rule of Thumb:**
    *   For simple string literals or few concatenations, `String` is fine.
    *   For string manipulation in a **single-threaded** environment (most common case), use **`StringBuilder`**.
    *   For string manipulation in a **multi-threaded** environment where concurrent access to the same string builder is expected, use **`StringBuffer`**.

In modern Java, `StringBuilder` is almost always preferred over `StringBuffer` in single-threaded contexts due to its superior performance. When thread safety is needed, consider other concurrent constructs before immediately reaching for `StringBuffer`.
