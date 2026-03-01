# STL Complexity Cheatsheet

Quick reference for time complexities of common STL operations.

## Sequence Containers

### std::vector

| Operation | Complexity |
|-----------|------------|
| `push_back` | Amortized O(1) |
| `pop_back` | O(1) |
| `insert` (at position) | O(n) |
| `erase` (at position) | O(n) |
| `operator[]` | O(1) |
| `at()` | O(1) |
| `front()` / `back()` | O(1) |
| `size()` | O(1) |
| `clear()` | O(n) |

### std::deque

| Operation | Complexity |
|-----------|------------|
| `push_back` / `push_front` | O(1) |
| `pop_back` / `pop_front` | O(1) |
| `insert` (middle) | O(n) |
| `operator[]` | O(1) |
| `front()` / `back()` | O(1) |

### std::list

| Operation | Complexity |
|-----------|------------|
| `push_back` / `push_front` | O(1) |
| `pop_back` / `pop_front` | O(1) |
| `insert` (at iterator) | O(1) |
| `erase` (at iterator) | O(1) |
| Access by index | O(n) |
| `splice` | O(1) |

## Associative Containers (Tree-based)

### std::set / std::map

| Operation | Complexity |
|-----------|------------|
| `insert` | O(log n) |
| `erase` | O(log n) |
| `find` | O(log n) |
| `count` | O(log n) |
| `lower_bound` / `upper_bound` | O(log n) |

### std::multiset / std::multimap

Same as set/map, but `count` is O(log n + k) where k is number of matches.

## Unordered Containers (Hash-based)

### std::unordered_set / std::unordered_map

| Operation | Average | Worst |
|-----------|---------|-------|
| `insert` | O(1) | O(n) |
| `erase` | O(1) | O(n) |
| `find` | O(1) | O(n) |
| `count` | O(1) | O(n) |

## Container Adapters

### std::stack / std::queue

| Operation | Complexity |
|-----------|------------|
| `push` | O(1) |
| `pop` | O(1) |
| `top` / `front` / `back` | O(1) |

### std::priority_queue

| Operation | Complexity |
|-----------|------------|
| `push` | O(log n) |
| `pop` | O(log n) |
| `top` | O(1) |
| Build from n elements | O(n) |

## std::string

| Operation | Complexity |
|-----------|------------|
| `operator[]` | O(1) |
| `push_back` | Amortized O(1) |
| `append` | O(m) where m = added length |
| `substr` | O(k) where k = substring length |
| `find` | O(n*m) |
| `compare` | O(min(n, m)) |

## Algorithms

| Algorithm | Complexity |
|-----------|------------|
| `std::sort` | O(n log n) |
| `std::stable_sort` | O(n log n) |
| `std::partial_sort` | O(n log k) |
| `std::nth_element` | O(n) average |
| `std::binary_search` | O(log n) |
| `std::lower_bound` | O(log n) |
| `std::find` | O(n) |
| `std::count` | O(n) |
| `std::reverse` | O(n) |
| `std::unique` | O(n) |
| `std::min_element` | O(n) |
| `std::accumulate` | O(n) |
| `std::copy` | O(n) |

## Space Complexity

| Container | Space |
|-----------|-------|
| `vector` | O(n) |
| `deque` | O(n) |
| `list` | O(n) |
| `set` / `map` | O(n) |
| `unordered_set` / `unordered_map` | O(n) |
| `priority_queue` | O(n) |

## Choosing the Right Container

| Need | Use |
|------|-----|
| Dynamic array | `vector` |
| Fast front/back ops | `deque` |
| Fast insert anywhere | `list` |
| Sorted unique keys | `set` |
| Key-value, sorted | `map` |
| Fast lookup by key | `unordered_map` |
| LIFO | `stack` |
| FIFO | `queue` |
| Priority access | `priority_queue` |
