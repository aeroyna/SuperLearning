# CopyOnWriteArrayList

A specialized list designed for specific concurrency scenarios where **Reads vastly outnumber Writes**.

## 1. The Mechanism: Copy-On-Write
*   **Internal Storage:** An immutable array.
*   **Reading:** No locking. No synchronization. Readers just look at the current array reference. Extremely fast.
*   **Writing:** 
    1.  Acquire a lock (so only one writer at a time).
    2.  **Copy** the entire underlying array to a new array of size `N + 1`.
    3.  Write the new element to the new array.
    4.  Atomically swap the array reference to point to the new array.
    5.  Release lock.

## 2. Implications
*   **Snapshot Iterators:** When you create an Iterator, it holds a reference to the array *as it existed* at that moment.
    *   If another thread modifies the list, the iterator continues traversing the *old* array.
    *   **Benefit:** No `ConcurrentModificationException`. No need to lock during iteration.
    *   **Cost:** "Eventual Consistency". The iterator might see stale data.

## 3. When to Use?
*   **Good:** Listeners/Observers lists (you iterate often to notify, add/remove listeners rarely). Configuration lists.
*   **Bad:** General purpose lists with frequent updates. The cost of array copying (`O(N)` memory and CPU) on every `add()` is catastrophic for large lists.