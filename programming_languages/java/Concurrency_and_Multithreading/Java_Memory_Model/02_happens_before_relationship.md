# The Happens-Before Relationship

This is the central rule of the JMM. It is a guarantee that memory writes by one specific statement are visible to another specific statement.

**Definition:** If Action A *happens-before* Action B, then the JMM guarantees that all memory changes made by A are visible to B, and A completes before B starts.

## The Rules
1.  **Program Order Rule:** Within a single thread, execution happens in order.
2.  **Monitor Lock Rule:** Unlocking a monitor *happens-before* every subsequent lock on that same monitor.
    *   *Meaning:* Everything Thread A did before releasing the lock is visible to Thread B when it gets the lock.
3.  **Volatile Variable Rule:** A write to a `volatile` field *happens-before* every subsequent read of that same field.
4.  **Thread Start Rule:** Calling `thread.start()` *happens-before* any action in the started thread.
5.  **Thread Join Rule:** All actions in a thread *happen-before* any other thread returns from a `join()` on that thread.

**Transitivity:** If A hb B, and B hb C, then A hb C.