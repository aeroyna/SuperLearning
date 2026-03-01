# List Interface: Deep Dive

## 1. ArrayList Internals

*   **Backing:** `Object[] elementData`.
*   **Random Access:** The CPU loves arrays. They are contiguous in memory, meaning pre-fetching works perfectly. Iterating an `ArrayList` is significantly faster than `LinkedList` due to **Cache Locality**.

### Resizing Mechanics
*   **Growth:** When full, capacity grows by **50%** (`oldCapacity + (oldCapacity >> 1)`).
*   **Shrinking:** `ArrayList` does **not** shrink automatically when elements are removed. You must call `trimToSize()` manually if you want to reclaim memory.

## 2. LinkedList Internals

*   **Node:** Requires a wrapper object (`Node`) for every element.
    *   32-bit JVM: 12 bytes header + 4 data + 4 next + 4 prev = 24 bytes per node overhead.
    *   64-bit JVM: Significantly higher overhead.
*   **Locality:** Nodes are allocated randomly in Heap. Traversing requires jumping around memory pages, causing CPU Cache Misses.

## 3. The "Vector" Legacy
`Vector` increments capacity by **100%** (doubling) and synchronizes every method. It is obsolete.

## 4. SubLists
`list.subList(from, to)` does **not** copy data. It returns a **View** (wrapper) over the original list.
*   *Danger:* Modifying the sublist modifies the original. Modifying the original structurally (add/remove) makes the sublist undefined/throw exception.