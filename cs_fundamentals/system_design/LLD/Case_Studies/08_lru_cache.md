# LRU Cache Design

## Problem Statement

Design a Least Recently Used (LRU) cache with O(1) get and put operations.

---

## Requirements

### Functional
- Get value by key in O(1)
- Put key-value pair in O(1)
- Evict least recently used item when capacity is reached
- Update access order on get/put

### Non-Functional
- Thread-safe operations
- Memory efficient
- Configurable capacity

---

## Data Structure

The LRU cache uses:
- **HashMap**: For O(1) key lookup
- **Doubly Linked List**: For O(1) removal and insertion

```
┌─────────────────────────────────────────────────────────┐
│                     HashMap                              │
│  key1 ──► node1                                          │
│  key2 ──► node2                                          │
│  key3 ──► node3                                          │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
┌──────┐   ┌──────┐   ┌──────┐   ┌──────┐   ┌──────┐
│ HEAD │◄─►│node1 │◄─►│node2 │◄─►│node3 │◄─►│ TAIL │
│(dummy)│   │ MRU  │   │      │   │ LRU  │   │(dummy)│
└──────┘   └──────┘   └──────┘   └──────┘   └──────┘
```

---

## Implementation

```python
from typing import Dict, Optional, Any, Generic, TypeVar
from dataclasses import dataclass
import threading
from collections import OrderedDict

K = TypeVar('K')
V = TypeVar('V')

@dataclass
class Node(Generic[K, V]):
    """Doubly linked list node"""
    key: K
    value: V
    prev: Optional['Node'] = None
    next: Optional['Node'] = None

class DoublyLinkedList(Generic[K, V]):
    """Doubly linked list with dummy head and tail"""

    def __init__(self):
        self.head = Node(None, None)  # Dummy head (MRU side)
        self.tail = Node(None, None)  # Dummy tail (LRU side)
        self.head.next = self.tail
        self.tail.prev = self.head
        self._size = 0

    def add_to_front(self, node: Node) -> None:
        """Add node right after head (most recently used)"""
        node.prev = self.head
        node.next = self.head.next
        self.head.next.prev = node
        self.head.next = node
        self._size += 1

    def remove(self, node: Node) -> None:
        """Remove a node from the list"""
        node.prev.next = node.next
        node.next.prev = node.prev
        node.prev = None
        node.next = None
        self._size -= 1

    def remove_last(self) -> Optional[Node]:
        """Remove and return the last node (least recently used)"""
        if self._size == 0:
            return None
        last = self.tail.prev
        self.remove(last)
        return last

    def move_to_front(self, node: Node) -> None:
        """Move existing node to front (mark as most recently used)"""
        self.remove(node)
        self.add_to_front(node)

    @property
    def size(self) -> int:
        return self._size

class LRUCache(Generic[K, V]):
    """LRU Cache with O(1) operations"""

    def __init__(self, capacity: int):
        if capacity <= 0:
            raise ValueError("Capacity must be positive")

        self.capacity = capacity
        self.cache: Dict[K, Node[K, V]] = {}
        self.list = DoublyLinkedList[K, V]()
        self._lock = threading.RLock()

        # Statistics
        self.hits = 0
        self.misses = 0

    def get(self, key: K) -> Optional[V]:
        """Get value by key, returns None if not found"""
        with self._lock:
            if key not in self.cache:
                self.misses += 1
                return None

            node = self.cache[key]
            self.list.move_to_front(node)
            self.hits += 1
            return node.value

    def put(self, key: K, value: V) -> None:
        """Add or update key-value pair"""
        with self._lock:
            if key in self.cache:
                # Update existing
                node = self.cache[key]
                node.value = value
                self.list.move_to_front(node)
            else:
                # Add new
                if self.list.size >= self.capacity:
                    # Evict LRU
                    lru = self.list.remove_last()
                    if lru:
                        del self.cache[lru.key]

                node = Node(key, value)
                self.cache[key] = node
                self.list.add_to_front(node)

    def delete(self, key: K) -> bool:
        """Remove a key from the cache"""
        with self._lock:
            if key not in self.cache:
                return False

            node = self.cache[key]
            self.list.remove(node)
            del self.cache[key]
            return True

    def clear(self) -> None:
        """Clear all entries"""
        with self._lock:
            self.cache.clear()
            self.list = DoublyLinkedList()

    def contains(self, key: K) -> bool:
        """Check if key exists (doesn't update access order)"""
        return key in self.cache

    def size(self) -> int:
        """Current number of entries"""
        return len(self.cache)

    def get_stats(self) -> Dict[str, Any]:
        """Get cache statistics"""
        total = self.hits + self.misses
        hit_rate = (self.hits / total * 100) if total > 0 else 0
        return {
            "capacity": self.capacity,
            "size": len(self.cache),
            "hits": self.hits,
            "misses": self.misses,
            "hit_rate": f"{hit_rate:.1f}%"
        }

    def __repr__(self) -> str:
        items = []
        node = self.list.head.next
        while node != self.list.tail:
            items.append(f"{node.key}:{node.value}")
            node = node.next
        return f"LRUCache([{', '.join(items)}])"
```

---

## Python OrderedDict Implementation

```python
class LRUCacheSimple(Generic[K, V]):
    """Simplified LRU Cache using OrderedDict"""

    def __init__(self, capacity: int):
        self.capacity = capacity
        self.cache: OrderedDict[K, V] = OrderedDict()

    def get(self, key: K) -> Optional[V]:
        if key not in self.cache:
            return None
        # Move to end (most recently used)
        self.cache.move_to_end(key)
        return self.cache[key]

    def put(self, key: K, value: V) -> None:
        if key in self.cache:
            self.cache.move_to_end(key)
        self.cache[key] = value

        if len(self.cache) > self.capacity:
            # Remove first item (least recently used)
            self.cache.popitem(last=False)
```

---

## TTL-Enabled LRU Cache

```python
from time import time
from typing import Tuple

class TTLLRUCache(Generic[K, V]):
    """LRU Cache with Time-To-Live expiration"""

    def __init__(self, capacity: int, default_ttl: int = 3600):
        self.capacity = capacity
        self.default_ttl = default_ttl  # seconds
        self.cache: Dict[K, Tuple[Node[K, V], float]] = {}  # key -> (node, expiry)
        self.list = DoublyLinkedList[K, V]()
        self._lock = threading.RLock()

    def get(self, key: K) -> Optional[V]:
        with self._lock:
            if key not in self.cache:
                return None

            node, expiry = self.cache[key]

            # Check if expired
            if time() > expiry:
                self._remove(key)
                return None

            self.list.move_to_front(node)
            return node.value

    def put(self, key: K, value: V, ttl: int = None) -> None:
        with self._lock:
            ttl = ttl or self.default_ttl
            expiry = time() + ttl

            if key in self.cache:
                node, _ = self.cache[key]
                node.value = value
                self.cache[key] = (node, expiry)
                self.list.move_to_front(node)
            else:
                # Cleanup expired entries first
                self._cleanup_expired()

                if self.list.size >= self.capacity:
                    lru = self.list.remove_last()
                    if lru:
                        del self.cache[lru.key]

                node = Node(key, value)
                self.cache[key] = (node, expiry)
                self.list.add_to_front(node)

    def _remove(self, key: K) -> None:
        if key in self.cache:
            node, _ = self.cache[key]
            self.list.remove(node)
            del self.cache[key]

    def _cleanup_expired(self) -> None:
        """Remove all expired entries"""
        now = time()
        expired = [k for k, (_, exp) in self.cache.items() if now > exp]
        for key in expired:
            self._remove(key)
```

---

## Thread-Safe LRU Cache with Read-Write Lock

```python
from threading import RLock
from contextlib import contextmanager

class ReadWriteLock:
    """Read-Write lock allowing multiple readers or single writer"""

    def __init__(self):
        self._read_ready = threading.Condition(threading.RLock())
        self._readers = 0

    @contextmanager
    def read_lock(self):
        with self._read_ready:
            self._readers += 1
        try:
            yield
        finally:
            with self._read_ready:
                self._readers -= 1
                if self._readers == 0:
                    self._read_ready.notify_all()

    @contextmanager
    def write_lock(self):
        with self._read_ready:
            while self._readers > 0:
                self._read_ready.wait()
            yield

class ConcurrentLRUCache(Generic[K, V]):
    """Thread-safe LRU Cache with read-write lock"""

    def __init__(self, capacity: int):
        self.capacity = capacity
        self.cache: Dict[K, Node[K, V]] = {}
        self.list = DoublyLinkedList[K, V]()
        self._rw_lock = ReadWriteLock()
        self._write_lock = threading.Lock()

    def get(self, key: K) -> Optional[V]:
        # Read operation - multiple readers allowed
        with self._rw_lock.read_lock():
            if key not in self.cache:
                return None
            node = self.cache[key]
            value = node.value

        # Update access order requires write lock
        with self._write_lock:
            if key in self.cache:
                self.list.move_to_front(self.cache[key])

        return value

    def put(self, key: K, value: V) -> None:
        with self._rw_lock.write_lock():
            if key in self.cache:
                node = self.cache[key]
                node.value = value
                self.list.move_to_front(node)
            else:
                if self.list.size >= self.capacity:
                    lru = self.list.remove_last()
                    if lru:
                        del self.cache[lru.key]

                node = Node(key, value)
                self.cache[key] = node
                self.list.add_to_front(node)
```

---

## LRU Cache with Eviction Callback

```python
from typing import Callable

class LRUCacheWithCallback(Generic[K, V]):
    """LRU Cache that calls a function when items are evicted"""

    def __init__(self, capacity: int,
                 on_evict: Callable[[K, V], None] = None):
        self.capacity = capacity
        self.on_evict = on_evict
        self.cache: Dict[K, Node[K, V]] = {}
        self.list = DoublyLinkedList[K, V]()

    def put(self, key: K, value: V) -> None:
        if key in self.cache:
            node = self.cache[key]
            old_value = node.value
            node.value = value
            self.list.move_to_front(node)
        else:
            if self.list.size >= self.capacity:
                lru = self.list.remove_last()
                if lru:
                    del self.cache[lru.key]
                    # Call eviction callback
                    if self.on_evict:
                        self.on_evict(lru.key, lru.value)

            node = Node(key, value)
            self.cache[key] = node
            self.list.add_to_front(node)

    def get(self, key: K) -> Optional[V]:
        if key not in self.cache:
            return None
        node = self.cache[key]
        self.list.move_to_front(node)
        return node.value
```

---

## Usage Example

```python
def demo_lru_cache():
    print("=== LRU Cache Demo ===\n")

    cache = LRUCache[str, int](capacity=3)

    # Add items
    cache.put("a", 1)
    cache.put("b", 2)
    cache.put("c", 3)
    print(f"After adding a, b, c: {cache}")

    # Access 'a' - moves to front
    cache.get("a")
    print(f"After accessing 'a': {cache}")

    # Add 'd' - evicts LRU ('b')
    cache.put("d", 4)
    print(f"After adding 'd': {cache}")

    # Check 'b' was evicted
    print(f"Get 'b': {cache.get('b')}")  # None

    # Update existing
    cache.put("a", 10)
    print(f"After updating 'a': {cache}")

    # Stats
    print(f"\nStats: {cache.get_stats()}")

def demo_ttl_cache():
    print("\n=== TTL LRU Cache Demo ===\n")
    import time

    cache = TTLLRUCache[str, str](capacity=5, default_ttl=2)

    cache.put("key1", "value1")
    cache.put("key2", "value2", ttl=1)  # 1 second TTL

    print(f"key1: {cache.get('key1')}")
    print(f"key2: {cache.get('key2')}")

    print("\nWaiting 1.5 seconds...")
    time.sleep(1.5)

    print(f"key1: {cache.get('key1')}")  # Still valid
    print(f"key2: {cache.get('key2')}")  # Expired

def demo_eviction_callback():
    print("\n=== Eviction Callback Demo ===\n")

    def on_evict(key, value):
        print(f"  Evicted: {key} = {value}")

    cache = LRUCacheWithCallback[str, int](capacity=2, on_evict=on_evict)

    cache.put("a", 1)
    cache.put("b", 2)
    print("Adding 'c' (will evict 'a'):")
    cache.put("c", 3)

if __name__ == "__main__":
    demo_lru_cache()
    demo_ttl_cache()
    demo_eviction_callback()
```

---

## Complexity Analysis

| Operation | Time | Space |
|-----------|------|-------|
| get() | O(1) | O(1) |
| put() | O(1) | O(1) |
| delete() | O(1) | O(1) |
| Overall | - | O(capacity) |

---

## Design Patterns Used

| Pattern | Usage |
|---------|-------|
| **Composite** | Node structure |
| **Observer** | Eviction callbacks |
| **Strategy** | Different eviction policies |

---

**Tags**: #lld #case-study #cache #lru #data-structure
