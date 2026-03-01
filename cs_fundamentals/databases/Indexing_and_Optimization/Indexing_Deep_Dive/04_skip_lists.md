# Skip Lists

## Introduction

Skip lists are probabilistic data structures that provide O(log n) search, insert, and delete operations. Invented by William Pugh in 1989, they offer a simpler alternative to balanced trees and are widely used in databases for in-memory indexing, particularly in MemTables of LSM trees.

## Core Concept

```
┌─────────────────────────────────────────────────────────────┐
│  Skip List: Linked list with express lanes                  │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Analogy: Express trains vs local trains                    │
│                                                              │
│  Regular Linked List (all stops):                           │
│  HEAD → 3 → 6 → 7 → 9 → 12 → 19 → 21 → 25 → 26 → NIL       │
│  Search for 19: Visit 8 nodes                               │
│                                                              │
│  Skip List (express + local):                                │
│  L3: HEAD ────────────────→ 12 ──────────────────→ NIL     │
│  L2: HEAD ─────→ 6 ────────→ 12 ─────→ 21 ───────→ NIL     │
│  L1: HEAD ─→ 3 → 6 ─→ 9 ───→ 12 → 19 → 21 ─→ 26 → NIL     │
│  L0: HEAD → 3 → 6 → 7 → 9 → 12 → 19 → 21 → 25 → 26 → NIL   │
│                                                              │
│  Search for 19:                                              │
│  L3: 12 < 19, move right → NIL, drop down                   │
│  L2: 21 > 19, drop down                                      │
│  L1: 19 = 19, FOUND!                                         │
│  Nodes visited: 4                                            │
└─────────────────────────────────────────────────────────────┘
```

## Structure

### Node Layout

```
┌─────────────────────────────────────────────────────────────┐
│                      Skip List Node                          │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  struct SkipListNode {                                       │
│      Key key;                                                │
│      Value value;                                            │
│      int height;           // Number of levels              │
│      Node* forward[];      // Array of forward pointers     │
│  }                                                           │
│                                                              │
│  Memory Layout:                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  Key: "user:123"                                     │    │
│  │  Value: {data...}                                    │    │
│  │  Height: 4                                           │    │
│  │  Forward Pointers:                                   │    │
│  │    [0] → next node at level 0                       │    │
│  │    [1] → next node at level 1                       │    │
│  │    [2] → next node at level 2                       │    │
│  │    [3] → next node at level 3                       │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Header Node:                                                │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  Key: -∞ (sentinel)                                  │    │
│  │  Height: MAX_LEVEL (e.g., 32)                        │    │
│  │  Forward: [ptr, ptr, ptr, ... ptr]                   │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

### Level Distribution

```
┌─────────────────────────────────────────────────────────────┐
│  Probabilistic Level Assignment                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Each node gets a random level using:                        │
│    P(level >= L) = p^L  where p = 0.5 (typical) or 0.25    │
│                                                              │
│  With p = 0.5:                                               │
│    Level 0: 100% of nodes                                    │
│    Level 1: 50% of nodes                                     │
│    Level 2: 25% of nodes                                     │
│    Level 3: 12.5% of nodes                                   │
│    Level 4: 6.25% of nodes                                   │
│    ...                                                       │
│                                                              │
│  Visual Distribution (16 nodes):                             │
│  L4: ■                                                       │
│  L3: ■ ■                                                     │
│  L2: ■ ■ ■ ■                                                 │
│  L1: ■ ■ ■ ■ ■ ■ ■ ■                                         │
│  L0: ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■                         │
│                                                              │
│  Expected height of tallest node: log_{1/p}(n)              │
│  For n=1M, p=0.5: ~20 levels                                │
└─────────────────────────────────────────────────────────────┘
```

## Operations

### Search Algorithm

```
FUNCTION search(list, searchKey):
    node = list.header

    // Start from highest level and work down
    FOR level = list.currentLevel DOWN TO 0:
        // Move forward while next key is smaller
        WHILE node.forward[level] != NIL AND
              node.forward[level].key < searchKey:
            node = node.forward[level]

    // Move to level 0 candidate
    node = node.forward[0]

    IF node != NIL AND node.key == searchKey:
        RETURN node.value
    ELSE:
        RETURN NOT_FOUND
```

```
Search Example: Find key = 21

L4: HEAD ─────────────────────────────────────────→ NIL
         ↓ (21 not found at L4, drop down)
L3: HEAD ──────────────→ 12 ─────────────────────→ NIL
                         ↓ (12 < 21, move right)
                         └→ NIL (drop down)
L2: HEAD ────→ 6 ───────→ 12 ──────→ 21 ─────────→ NIL
                                     ↓ (21 = 21, found!)

Path: HEAD(L4) → HEAD(L3) → 12(L3) → 12(L2) → 21(L2)
Comparisons: ~4 (vs 7 in linear search)
```

### Insert Algorithm

```
FUNCTION insert(list, key, value):
    // Track nodes that need updating
    update = array of MAX_LEVEL nodes
    node = list.header

    // Find position and record predecessors
    FOR level = list.currentLevel DOWN TO 0:
        WHILE node.forward[level] != NIL AND
              node.forward[level].key < key:
            node = node.forward[level]
        update[level] = node

    node = node.forward[0]

    IF node != NIL AND node.key == key:
        // Key exists, update value
        node.value = value
    ELSE:
        // Generate random level
        newLevel = randomLevel()

        IF newLevel > list.currentLevel:
            FOR level = list.currentLevel + 1 TO newLevel:
                update[level] = list.header
            list.currentLevel = newLevel

        // Create new node
        newNode = createNode(key, value, newLevel)

        // Insert at each level
        FOR level = 0 TO newLevel:
            newNode.forward[level] = update[level].forward[level]
            update[level].forward[level] = newNode

FUNCTION randomLevel():
    level = 0
    WHILE random() < p AND level < MAX_LEVEL - 1:
        level = level + 1
    RETURN level
```

### Insert Visualization

```
Insert key = 17:

Before:
L3: HEAD ──────────────→ 12 ────────────────→ 25 ──→ NIL
L2: HEAD ────→ 6 ───────→ 12 ───────→ 21 ───→ 25 ──→ NIL
L1: HEAD ─→ 3 → 6 → 9 ───→ 12 → 19 → 21 ─────→ 25 ──→ NIL
L0: HEAD → 3 → 6 → 7 → 9 → 12 → 19 → 21 → 22 → 25 → NIL

Step 1: Random level for 17 = 2 (coin flips: H, H, T)

Step 2: Find predecessors at each level
    update[0] = 12 (node before 17 at level 0)
    update[1] = 12 (node before 17 at level 1)
    update[2] = 12 (node before 17 at level 2)

Step 3: Link new node

After:
L3: HEAD ──────────────→ 12 ────────────────→ 25 ──→ NIL
L2: HEAD ────→ 6 ───────→ 12 ──→ 17 → 21 ───→ 25 ──→ NIL
L1: HEAD ─→ 3 → 6 → 9 ───→ 12 → 17 → 19 → 21 → 25 ──→ NIL
L0: HEAD → 3 → 6 → 7 → 9 → 12 → 17 → 19 → 21 → 22 → 25 → NIL
                               ↑
                           New node
```

### Delete Algorithm

```
FUNCTION delete(list, key):
    update = array of MAX_LEVEL nodes
    node = list.header

    // Find node and predecessors
    FOR level = list.currentLevel DOWN TO 0:
        WHILE node.forward[level] != NIL AND
              node.forward[level].key < key:
            node = node.forward[level]
        update[level] = node

    node = node.forward[0]

    IF node != NIL AND node.key == key:
        // Unlink at each level
        FOR level = 0 TO list.currentLevel:
            IF update[level].forward[level] != node:
                BREAK
            update[level].forward[level] = node.forward[level]

        // Decrease level if necessary
        WHILE list.currentLevel > 0 AND
              list.header.forward[list.currentLevel] == NIL:
            list.currentLevel = list.currentLevel - 1

        FREE node
        RETURN TRUE

    RETURN FALSE
```

## Complexity Analysis

```
┌─────────────────────────────────────────────────────────────┐
│                 Skip List Complexity                         │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Operation        Average Case    Worst Case                │
│  ─────────────────────────────────────────────────────────  │
│  Search           O(log n)        O(n)*                     │
│  Insert           O(log n)        O(n)*                     │
│  Delete           O(log n)        O(n)*                     │
│  Space            O(n)            O(n log n)*               │
│                                                              │
│  * Worst case is extremely rare due to randomization        │
│                                                              │
│  Expected Comparisons per Search:                            │
│    With p = 0.5: (log₂n)/2 + O(1) ≈ log₂n comparisons      │
│    With p = 0.25: (log₄n) + O(1) ≈ 0.5 log₂n comparisons   │
│                                                              │
│  Space Overhead:                                             │
│    Expected pointers per node: 1/(1-p)                       │
│    With p = 0.5: 2 pointers per node average                │
│    With p = 0.25: 1.33 pointers per node average            │
└─────────────────────────────────────────────────────────────┘
```

## Implementation

### Complete Implementation

```python
import random
from typing import Optional, TypeVar, Generic

K = TypeVar('K')
V = TypeVar('V')

class SkipListNode(Generic[K, V]):
    def __init__(self, key: K, value: V, level: int):
        self.key = key
        self.value = value
        self.forward: list[Optional[SkipListNode[K, V]]] = [None] * (level + 1)

class SkipList(Generic[K, V]):
    def __init__(self, max_level: int = 16, p: float = 0.5):
        self.max_level = max_level
        self.p = p
        self.level = 0
        self.header: SkipListNode[K, V] = SkipListNode(None, None, max_level)
        self.size = 0

    def _random_level(self) -> int:
        level = 0
        while random.random() < self.p and level < self.max_level:
            level += 1
        return level

    def search(self, key: K) -> Optional[V]:
        node = self.header
        for i in range(self.level, -1, -1):
            while node.forward[i] and node.forward[i].key < key:
                node = node.forward[i]

        node = node.forward[0]
        if node and node.key == key:
            return node.value
        return None

    def insert(self, key: K, value: V) -> None:
        update = [None] * (self.max_level + 1)
        node = self.header

        for i in range(self.level, -1, -1):
            while node.forward[i] and node.forward[i].key < key:
                node = node.forward[i]
            update[i] = node

        node = node.forward[0]

        if node and node.key == key:
            node.value = value
            return

        new_level = self._random_level()

        if new_level > self.level:
            for i in range(self.level + 1, new_level + 1):
                update[i] = self.header
            self.level = new_level

        new_node = SkipListNode(key, value, new_level)

        for i in range(new_level + 1):
            new_node.forward[i] = update[i].forward[i]
            update[i].forward[i] = new_node

        self.size += 1

    def delete(self, key: K) -> bool:
        update = [None] * (self.max_level + 1)
        node = self.header

        for i in range(self.level, -1, -1):
            while node.forward[i] and node.forward[i].key < key:
                node = node.forward[i]
            update[i] = node

        node = node.forward[0]

        if node and node.key == key:
            for i in range(self.level + 1):
                if update[i].forward[i] != node:
                    break
                update[i].forward[i] = node.forward[i]

            while self.level > 0 and self.header.forward[self.level] is None:
                self.level -= 1

            self.size -= 1
            return True

        return False

    def range_query(self, start: K, end: K) -> list[tuple[K, V]]:
        """Return all key-value pairs where start <= key <= end."""
        result = []
        node = self.header

        # Find start position
        for i in range(self.level, -1, -1):
            while node.forward[i] and node.forward[i].key < start:
                node = node.forward[i]

        node = node.forward[0]

        # Collect all keys in range
        while node and node.key <= end:
            result.append((node.key, node.value))
            node = node.forward[0]

        return result
```

## Concurrent Skip Lists

### Lock-Free Implementation Concepts

```
┌─────────────────────────────────────────────────────────────┐
│              Concurrent Skip List Techniques                 │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Lock-Based (Simple):                                     │
│     - One lock per node                                      │
│     - Lock predecessors and target during modification       │
│     - High contention for popular paths                      │
│                                                              │
│  2. Lock-Free (CAS-based):                                   │
│     - No locks, use Compare-And-Swap                         │
│     - Mark nodes before deletion (logical delete)            │
│     - Help complete ongoing operations                       │
│                                                              │
│  Marking Scheme:                                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ Forward pointer with mark bit:                        │   │
│  │   [0x00001234|0] - unmarked, valid pointer           │   │
│  │   [0x00001234|1] - marked for deletion               │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                              │
│  Delete Process:                                             │
│  1. Mark forward pointers (logical delete)                   │
│  2. CAS to unlink node (physical delete)                    │
│  3. Other threads help complete if they see marked nodes    │
└─────────────────────────────────────────────────────────────┘
```

### Java ConcurrentSkipListMap

```java
// Java's concurrent skip list
import java.util.concurrent.ConcurrentSkipListMap;
import java.util.concurrent.ConcurrentSkipListSet;

// Usage
ConcurrentSkipListMap<String, User> users = new ConcurrentSkipListMap<>();

// Thread-safe operations
users.put("user:123", new User("Alice"));
users.put("user:456", new User("Bob"));

// Range queries
NavigableMap<String, User> range = users.subMap("user:100", "user:200");

// Lock-free iteration (weakly consistent)
for (Map.Entry<String, User> entry : users.entrySet()) {
    // Safe to iterate while others modify
}
```

## Database Usage

### Redis Sorted Sets

```
┌─────────────────────────────────────────────────────────────┐
│              Redis Sorted Set Implementation                 │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Redis ZSET uses skip list + hash table:                    │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Skip List: Sorted by score                          │    │
│  │   L2: HEAD ──→ (1.0,A) ────────→ (5.0,D) ──→ NIL  │    │
│  │   L1: HEAD ──→ (1.0,A) → (3.0,C) → (5.0,D) → NIL  │    │
│  │   L0: HEAD → (1.0,A) → (2.0,B) → (3.0,C) → (5.0,D)│    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Hash Table: O(1) lookup by member                   │    │
│  │   "A" → node pointer                                │    │
│  │   "B" → node pointer                                │    │
│  │   "C" → node pointer                                │    │
│  │   "D" → node pointer                                │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Operations:                                                 │
│    ZADD: O(log n) - skip list insert                        │
│    ZRANK: O(log n) - skip list search with rank tracking   │
│    ZRANGE: O(log n + m) - skip list range                   │
│    ZSCORE: O(1) - hash table lookup                         │
└─────────────────────────────────────────────────────────────┘
```

```redis
# Redis sorted set commands
ZADD leaderboard 100 "player:1" 250 "player:2" 180 "player:3"

# Get rank (skip list traversal with span counting)
ZRANK leaderboard "player:2"  # Returns 2 (0-indexed)

# Get range by rank
ZRANGE leaderboard 0 9 WITHSCORES  # Top 10

# Get range by score
ZRANGEBYSCORE leaderboard 100 200 WITHSCORES
```

### MemTable Implementation (LevelDB/RocksDB)

```cpp
// Simplified MemTable skip list node
struct MemTableNode {
    // Key format: internal_key (user_key + seq_num + type)
    const char* key;
    size_t key_size;

    // Forward pointers
    std::atomic<MemTableNode*> next[1];  // Flexible array

    MemTableNode* Next(int level) {
        return next[level].load(std::memory_order_acquire);
    }

    void SetNext(int level, MemTableNode* node) {
        next[level].store(node, std::memory_order_release);
    }
};

// MemTable operations
class MemTable {
    SkipList<const char*, Comparator> table_;

    void Add(SequenceNumber seq, ValueType type,
             const Slice& key, const Slice& value) {
        // Encode key with sequence number
        char* buf = AllocateKey(key.size() + 8 + value.size());
        EncodeKey(buf, key, seq, type);
        memcpy(buf + key.size() + 8, value.data(), value.size());

        // Insert into skip list
        table_.Insert(buf);
    }

    bool Get(const LookupKey& key, std::string* value, Status* s) {
        SkipList::Iterator iter(&table_);
        iter.Seek(key.memtable_key());
        if (iter.Valid()) {
            // Check if key matches
            // Handle tombstones
        }
    }
};
```

## Comparison with Other Structures

```
┌─────────────────────────────────────────────────────────────┐
│          Skip List vs Balanced Trees                         │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Feature            Skip List         Balanced Tree         │
│  ─────────────────────────────────────────────────────────  │
│  Implementation     Simple            Complex               │
│  Search             O(log n)          O(log n)              │
│  Insert             O(log n)          O(log n)              │
│  Delete             O(log n)          O(log n)              │
│  Space              2n pointers avg   3n pointers           │
│  Cache locality     Poor              Better                │
│  Lock-free          Easier            Harder                │
│  Range queries      Excellent         Good                  │
│  Ordered iteration  Trivial           Requires stack        │
│                                                              │
│  When to use Skip List:                                      │
│  ✓ Need concurrent access                                   │
│  ✓ Simple implementation preferred                          │
│  ✓ Range queries important                                  │
│  ✓ Memory not critical                                       │
│                                                              │
│  When to use Balanced Tree:                                  │
│  ✓ Memory efficiency critical                               │
│  ✓ Cache performance matters                                │
│  ✓ Deterministic worst-case needed                          │
│  ✓ No concurrent access required                            │
└─────────────────────────────────────────────────────────────┘
```

## Key Takeaways

1. **Probabilistic balance** - No rotations needed, randomization provides balance
2. **Simple implementation** - Much easier than AVL or Red-Black trees
3. **Excellent for concurrency** - Lock-free algorithms are practical
4. **Used in production** - Redis sorted sets, LevelDB/RocksDB MemTables
5. **Good for range queries** - Sequential access at bottom level
6. **Tunable with p parameter** - Balance between levels and search time
