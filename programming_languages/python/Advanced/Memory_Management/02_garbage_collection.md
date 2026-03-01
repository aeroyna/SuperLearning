# Garbage Collection (GC)

Python's Garbage Collector (GC) exists purely to solve the **Reference Cycle** problem. It runs periodically to find and clean up objects that refer to each other but are no longer reachable from the root.

## Generational GC
CPython uses a **generational** approach. Objects are classified into three generations (0, 1, 2).

1.  **Generation 0**: Young objects. Most objects die young. GC scans this frequently.
2.  **Generation 1**: Objects that survive a Gen 0 scan.
3.  **Generation 2**: Old objects. Scanned rarely.

## How it Works
1.  **Detection**: The GC looks for containers (lists, dicts, classes) specifically, as they can cause cycles.
2.  **Thresholds**: When the number of allocations minus deallocations exceeds a 700 (default), a Gen 0 collection is triggered.
3.  **Promotion**: Surviving objects are promoted to the next generation.

## Controlling the GC
The `gc` module allows interaction.

```python
import gc

# Force a collection
gc.collect()

# Get stats
print(gc.get_stats())

# Disable GC (optimization for short scripts)
gc.disable()
```

### Debugging Leaks
You can set flags to see leaked objects.

```python
gc.set_debug(gc.DEBUG_LEAK)
```
