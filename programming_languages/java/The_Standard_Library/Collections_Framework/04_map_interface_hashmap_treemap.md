# Map Interface: HashMap and TreeMap Deep Dive

## 1. HashMap Internals

`HashMap` works on the principle of hashing. It maintains an array of `Node<K,V>` (buckets).

### 1.1 The `put(K,V)` Mechanics
1.  **Hash Calculation:** Calls `key.hashCode()`. It then applies a "perturbation function" (XOR shift) to the hash to ensure lower bits are randomized.
2.  **Index:** `index = (n - 1) & hash` (where n is array size). This bitwise AND is faster than modulo `%`.
3.  **Collision:** If the bucket is empty, insert. If occupied, traverse the Linked List (or Tree).
4.  **Replacement:** If `equals()` returns true, overwrite value.

### 1.2 Performance Tuning
*   **Initial Capacity (Default 16):** The size of the bucket array.
*   **Load Factor (Default 0.75):** When the map is 75% full, it **Resizes**.
*   **Resize (Rehash):**
    1.  Create new array (size * 2).
    2.  Re-calculate index for EVERY entry.
    3.  Move entries to new buckets.
    *   *Cost:* **O(n)**. Very expensive.
    *   *Tip:* `new HashMap<>(expectedSize / 0.75)` prevents resizing.

### 1.3 Treeification (Java 8+)
To prevent Hash DoS attacks (where all keys collide to index 0), `HashMap` changes structure dynamically.
*   **Threshold:** Conversion to a Red-Black Tree happens only if **two** conditions are met:
    1.  A single bucket's linked list exceeds **8 nodes** (`TREEIFY_THRESHOLD`).
    2.  **AND** the total capacity of the array is at least **64** (`MIN_TREEIFY_CAPACITY`).
*   **Fallback:** If the bucket is too long but the array is small (< 64), `HashMap` will **resize** the array instead of treeifying, hoping to spread the colliding keys into new buckets.
*   **Impact:** Worst-case performance improves from `O(n)` (List) to `O(log n)` (Tree).

## 2. TreeMap Internals

*   **Structure:** Red-Black Tree (Self-balancing Binary Search Tree).
*   **Contract:** Keys are ordered.
*   **Cost:** Every insertion requires re-balancing (color flips, rotations). Slower than HashMap constant time.
*   **Use Case:** Range queries. `subMap(from, to)`, `headMap()`, `tailMap()`.

## 3. LinkedHashMap
*   **Under the hood:** It is a HashMap, but every Node has `before` and `after` pointers effectively forming a generic doubly-linked list running through the map.
*   **Access Order:** Can be configured to move accessed elements to the end of the list (LRU Cache implementation).