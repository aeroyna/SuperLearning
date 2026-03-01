# Atomicity, Visibility, and Ordering

These are the three demons of concurrency.

## 1. Atomicity
Deals with "indivisible" operations.
*   **Safe:** Reads/writes of reference variables and most primitives (int, byte, boolean) are atomic.
*   **Unsafe:** `long` and `double` are 64-bit. On 32-bit JVMs, reading/writing them is *two* operations (two 32-bit moves). Thread A could write the first 32 bits, Thread B reads, seeing a corrupted "torn" value. (Use `volatile` long/double to fix this).
*   **Unsafe:** `count++`. It is Read-Modify-Write.

## 2. Visibility
Deals with "stale data".
*   Caused by CPU Caching.
*   **Fix:** `volatile`, `synchronized`, `final` (after constructor).

## 3. Ordering (Instruction Reordering)
The compiler (javac), JIT (HotSpot), and the CPU are all allowed to **reorder** instructions to optimize performance, as long as the single-threaded semantics remain the same (as-if-serial).

**Example:**
```java
x = 1;
y = 2;
```
The CPU might execute `y = 2` before `x = 1`. In a single thread, you can't tell. In multithreading, it matters.

**The "Double-Checked Locking" Bug:**
```java
// Broken Singleton
if (instance == null) {
    synchronized(Lock.class) {
        if (instance == null) {
            instance = new Singleton(); 
            // 1. Allocate memory
            // 2. Init object (run constructor)
            // 3. Assign ref to 'instance'
            // REORDERING: The JVM might do 1 -> 3 -> 2.
            // Thread A executes 3. 'instance' is now NON-NULL but NOT INITIALIZED.
            // Thread B enters, sees instance != null, returns broken object.
        }
    }
}
```
**Fix:** Make `instance` **`volatile`**. This prevents reordering across the write.