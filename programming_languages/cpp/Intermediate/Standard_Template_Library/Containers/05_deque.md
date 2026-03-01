# std::deque (Double-Ended Queue)

`std::deque` (pronounced "deck") is a double-ended queue that allows fast insertion and removal at both ends.

## Include Header

```cpp
#include <deque>
```

## Creating a Deque

```cpp
std::deque<int> d1;                    // Empty deque
std::deque<int> d2 = {1, 2, 3, 4, 5};  // Initializer list
std::deque<int> d3(5, 10);             // 5 elements with value 10
std::deque<int> d4(d2);                // Copy of d2
```

## Key Operations

### Adding Elements

```cpp
std::deque<int> d;

d.push_back(10);    // Add to end: [10]
d.push_back(20);    // [10, 20]
d.push_front(5);    // Add to front: [5, 10, 20]
d.push_front(1);    // [1, 5, 10, 20]

d.emplace_back(30); // Construct in place at end
d.emplace_front(0); // Construct in place at front
```

### Removing Elements

```cpp
d.pop_back();       // Remove from end
d.pop_front();      // Remove from front
d.erase(d.begin()); // Remove at iterator
d.clear();          // Remove all
```

### Accessing Elements

```cpp
std::deque<int> d = {1, 2, 3, 4, 5};

d[0];        // 1 (no bounds check)
d.at(0);     // 1 (with bounds check, throws if invalid)
d.front();   // 1 (first element)
d.back();    // 5 (last element)
```

## deque vs vector

| Operation | deque | vector |
|-----------|-------|--------|
| `push_back` | O(1) | Amortized O(1) |
| `push_front` | O(1) | O(n) |
| `pop_back` | O(1) | O(1) |
| `pop_front` | O(1) | O(n) |
| Random access `[]` | O(1) | O(1) |
| Insert middle | O(n) | O(n) |
| Memory | Chunks | Contiguous |

### When to Use deque Over vector

- Need fast insertion/removal at **both** ends
- Don't need guaranteed contiguous memory
- Implementing a queue or work-stealing deque

### When to Use vector Over deque

- Need contiguous memory (cache locality)
- Only need operations at **one end**
- Interfacing with C APIs expecting arrays

## Internal Structure

Unlike `vector`, `deque` stores elements in **chunks** (blocks):

```
deque internal structure:
┌───────────┐
│ Block Ptr │──→ [ elem, elem, elem ]
│ Block Ptr │──→ [ elem, elem, elem ]
│ Block Ptr │──→ [ elem, elem, elem ]
└───────────┘
```

This allows O(1) front insertion without moving elements.

## Complete Example

```cpp
#include <iostream>
#include <deque>

int main() {
    std::deque<std::string> tasks;

    // Add tasks at different priorities
    tasks.push_back("Low priority task");
    tasks.push_front("High priority task");  // Goes to front
    tasks.push_back("Normal task");

    std::cout << "Tasks:\n";
    for (const auto& task : tasks) {
        std::cout << "- " << task << "\n";
    }

    // Process highest priority
    std::cout << "\nProcessing: " << tasks.front() << "\n";
    tasks.pop_front();

    return 0;
}
```

## Key Takeaways

- O(1) insertion/removal at **both** ends
- O(1) random access
- Not contiguous in memory
- Iterators may be invalidated more easily than vector
- Use for double-ended operations, queues

## Common Interview Questions

> [!question]- When would you use deque over vector?
> When you need efficient insertion/removal at both ends, like implementing a sliding window or a work queue with priority insertion.

> [!question]- Is deque contiguous in memory?
> No. It uses chunks of memory. This is why `push_front` is O(1) but cache performance may be worse than vector.
