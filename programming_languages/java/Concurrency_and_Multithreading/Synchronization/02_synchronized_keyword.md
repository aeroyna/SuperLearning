# The Synchronized Keyword (Intrinsic Locks)

Java's built-in mechanism for mutual exclusion. It is based on the concept of an **Intrinsic Lock** or **Monitor Lock**.

## 1. How it works (Under the Hood)
Every Object in Java (and every Class) has an internal entity called a **Monitor**.
*   When a thread enters a `synchronized` block, it attempts to acquire ownership of that Monitor.
*   If the Monitor is free, the thread owns it and enters.
*   If the Monitor is owned by another thread, the calling thread is put into the **Entry Set** (BLOCKED state) and sleeps until the lock is released.
*   When the thread exits the block (or throws an exception), it releases the Monitor.

## 2. Syntax Variations

### Synchronized Instance Method
```java
public synchronized void method() {
    // code
}
```
*   **The Lock:** The instance (`this`).
*   **Scope:** Only one thread can execute *any* synchronized instance method of *this particular object* at a time.

### Synchronized Static Method
```java
public static synchronized void method() {
    // code
}
```
*   **The Lock:** The Class Object (`MyClass.class`).
*   **Scope:** Only one thread can execute this static method (or any other static synchronized method of this class) at a time.

### Synchronized Block (Fine-grained)
```java
public void method() {
    // Non-critical code (runs concurrently)
    
    synchronized(this) { 
        // Critical section (serialized)
    }
}
```
*   **The Lock:** Whatever object you pass in parentheses. Usually `this` or a dedicated lock object (`private final Object lock = new Object();`).

## 3. Reentrancy
Java Intrinsic Locks are **Reentrant**.
*   **Scenario:** Thread A holds the lock on Object X. It calls another method `foo()`, which is also `synchronized` on Object X.
*   **Result:** Thread A *can* enter `foo()` without blocking.
*   **Mechanism:** The lock keeps a "hold count".
    1.  Enter `synchronized`: Count 0 -> 1.
    2.  Enter nested `synchronized`: Count 1 -> 2.
    3.  Exit nested: Count 2 -> 1.
    4.  Exit outer: Count 1 -> 0 (Lock released).

## 4. Memory Visibility Guarantee
`synchronized` is not just about mutual exclusion (atomicity); it also guarantees **Memory Visibility**.
*   **Lock Acquisition:** When a thread enters a synchronized block, it is required to refresh its Working Memory (cache) from Main Memory. It sees the most up-to-date values.
*   **Lock Release:** When a thread exits a synchronized block, it must flush all changes from its Working Memory back to Main Memory.
*   **Result:** Everything done by Thread A *before* releasing the lock is visible to Thread B *after* it acquires the same lock.

## 5. Best Practices
1.  **Prefer Blocks over Methods:** Synchronizing the whole method is often overkill (performance penalty). Synchronize only the critical section.
2.  **Private Locks:** Synchronizing on `this` (instance methods) exposes your locking policy to the world. Any external code can do `synchronized(myObject) { ... }` and block your internal methods (Denial of Service).
    *   *Solution:* Use a private lock object:
        ```java
        private final Object lock = new Object();
        public void op() { synchronized(lock) { ... } }
        ```