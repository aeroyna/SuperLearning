# Race Conditions and Critical Sections: Deep Dive

This file dissects exactly how race conditions corrupt state at the CPU level.

## 1. The Anatomy of a Race Condition

Let's look at the classic `Counter` example at the Bytecode/CPU level.

```java
class Counter {
    int count = 0;
    void increment() {
        count++;
    }
}
```

**Bytecode View:**
When `count++` executes, the JVM (and effectively the CPU) performs:
1.  `GETFIELD`: Fetch `count` from Heap to Stack (CPU Register).
2.  `IADD`: Add 1 to the value in the Stack/Register.
3.  `PUTFIELD`: Write the new value back to Heap.

**The Failure Scenario:**
Assume `count` is initially `10`.

| Time | Thread A | Thread B | Value in Heap |
| :--- | :--- | :--- | :--- |
| T1 | Reads `10` | | 10 |
| T2 | | Reads `10` | 10 |
| T3 | Calculates `11` | | 10 |
| T4 | Writes `11` | | 11 |
| T5 | | Calculates `11` | 11 |
| T6 | | Writes `11` | **11** |

**Result:** The counter is `11`, but it should be `12`. Thread A's update was **overwritten** (lost).

## 2. Compound Actions vs Atomic Actions
*   **Atomic:** An action that happens "all at once" or "not at all". No other thread can see an intermediate state.
    *   *Example:* Reading a reference variable, reading a `volatile` long.
*   **Compound:** A sequence of operations that can be interrupted.
    *   *Example:* `i++`, `if (x == null) x = new X()`.

**Rule:** Every Compound Action acting on shared state must be guarded by synchronization to make it effectively atomic.

## 3. Critical Sections
The **Critical Section** is the specific lines of code where the shared resource is accessed.

**Identifying Critical Sections:**
1.  Look for **Shared Mutable State** (Fields of a class that are not `final`).
2.  Look for compound actions on that state.

**Minimizing Scope:**
Ideally, a critical section should be as short as possible.
*   *Bad:* Synchronize an entire method performing I/O.
*   *Good:* Calculate locally, then synchronize only the line that updates the shared map.