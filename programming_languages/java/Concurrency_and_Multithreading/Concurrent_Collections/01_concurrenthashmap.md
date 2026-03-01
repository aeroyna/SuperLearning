# ConcurrentHashMap

This is the workhorse of server-side Java. It allows concurrent reads and writes without locking the entire map.

## 1. Architecture: Lock Stripping (Java 5-7)
Instead of 1 lock for the whole map, it uses **16 locks (segments)** by default.
*   Keys are hashed to determine which segment they belong to.
*   Thread A writes to Segment 1. Thread B writes to Segment 2. **Both proceed purely concurrently.**
*   Only threads writing to the *same* segment block each other.

## 2. Architecture: CAS and Synchronized Buckets (Java 8+)
Java 8 rewrote CHM completely. Segments are gone.
*   **Reads (`get`):** Wait-free. No locks. Extremely fast. It relies on `volatile` reads of Node references.
*   **Writes (`put`):**
    1.  **Empty Bucket:** Uses **CAS (Compare-And-Swap)** to insert the first node. This is a CPU-level atomic instruction that succeeds only if the slot is still null. No lock used.
    2.  **Occupied Bucket:** Uses `synchronized` on the **Head Node** of that specific bucket (chain/tree). This provides the finest possible locking granularity (per-key-hash locking).

## 3. Atomic Composite Operations
`CHM` provides methods for atomic "Check-Then-Act" sequences that are impossible with standard maps without external locking.

```java
ConcurrentMap<String, Integer> map = new ConcurrentHashMap<>();

// Non-Atomic (Race Condition)
int old = map.get("key");
map.put("key", old + 1); 

// Atomic (Safe)
map.compute("key", (k, v) -> (v == null) ? 1 : v + 1);
map.putIfAbsent("key", 0);
```

## 4. Key Restrictions
*   **Nulls:** `ConcurrentHashMap` does **NOT** allow `null` keys or `null` values. (Unlike `HashMap`).
    *   *Why?* Ambiguity in concurrent scenarios. If `map.get(key)` returns `null`, does it mean the key is missing, or the value is null? In `HashMap`, you can check `contains()`. In a concurrent map, the map might change *between* `contains()` and `get()`, making the answer useless.