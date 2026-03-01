# StringBuilder and StringBuffer: Under the Hood

## 1. Internal Mechanics

Both classes extend `AbstractStringBuilder`.
*   **Backing Store:** A resizable `byte[]` array (Java 9+) or `char[]` (Java 8).
*   **Default Capacity:** 16 characters.
*   **Resizing Strategy:** When you append data that exceeds capacity:
    1.  **New Capacity:** `(current_capacity * 2) + 2`.
    2.  **Allocation:** A new array is allocated in the Heap.
    3.  **Copy:** `System.arraycopy()` copies old data to the new array.
    4.  **GC:** The old array is discarded.

*   **Performance Tip:** If you know the approximate size of the string beforehand, **always constructor-initialize with capacity**. This avoids the expensive resize/copy cycles.
    ```java
    StringBuilder sb = new StringBuilder(1024); // Pre-allocates 1KB
    ```

## 2. StringBuilder (Mutable, Non-Synchronized)
*   **Use Case:** 99% of string manipulation. Single-threaded context (e.g., inside a method).
*   **Speed:** fast. No locking overhead.

## 3. StringBuffer (Mutable, Synchronized)
*   **Synchronized:** Every method (`append`, `delete`) is `synchronized`.
*   **Use Case:** Historic legacy code or very specific multi-threaded text assembling (rare).
*   **Obsolete?** Mostly. Even in multi-threaded contexts, it's often better to have each thread build its own string locally and join them at the end, rather than contending on a shared `StringBuffer`.

## 4. Compiler Optimization
```java
String s = "Hello" + " " + "World";
```
The compiler automatically optimizes this single-line concatenation into a single String constant (if literals) or efficient concatenation logic. You do **not** need `StringBuilder` for simple one-liners. Only use `StringBuilder` for loops.
