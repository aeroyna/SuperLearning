# Working Set Model\n\n## Working Set Model

### Overview

Set of pages currently in use

### Mechanism

```c
// Page replacement
typedef struct {
    int page_num;
    int frame_num;
    unsigned long timestamp;
    bool referenced;
} Page;

int page_replacement(Page pages[], int num_pages) {
    // Algorithm-specific replacement logic
    return victim_page;
}
```

### Algorithms

Implementation of various strategies.

**Python Simulation**:
```python
def lru_replacement(pages, page_faults):
    cache = []
    for page in pages:
        if page not in cache:
            if len(cache) >= capacity:
                cache.pop(0)  # Remove LRU
            cache.append(page)
    return page_faults
```

### Performance Analysis

Hit ratio, fault rate, replacement frequency.

## Key Takeaways

1. Virtual memory allows execution of larger programs
2. Page replacement algorithms minimize page faults
3. Working set determines memory requirements
4. Thrashing occurs when working set exceeds memory

## Interview Focus

1. Explain demand paging
2. Compare page replacement algorithms
3. What causes thrashing? How to prevent?
4. Calculate page faults for given reference string
