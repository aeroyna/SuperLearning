# Data Structures

Data structures are the building blocks of efficient algorithms. Understanding not just what they do, but how they work internally—their memory layout, operational complexity, and implementation trade-offs—is essential for both solving problems and communicating solutions in interviews.

## Overview

Data structures fall into two broad categories:

1. **Linear Structures** - Elements arranged sequentially (Arrays, Linked Lists, Stacks, Queues)
2. **Non-Linear Structures** - Elements with hierarchical or network relationships (Trees, Graphs, Heaps)

Each structure optimizes for different operations. The art of DSA is matching the right structure to the problem constraints.

## Topics

### Linear Data Structures

- [3. Arrays and Strings](Arrays_and_Strings/00_arrays_and_strings.md) - Contiguous memory, O(1) access
- [4. Linked Lists](Linked_Lists/00_linked_lists.md) - Node-based, O(1) insertion
- [5. Stacks and Queues](Stacks_and_Queues/00_stacks_and_queues.md) - Restricted access patterns
- [6. Hash Tables](Hash_Tables/00_hash_tables.md) - Key-value mapping, O(1) average operations

### Non-Linear Data Structures

- [7. Trees](Trees/00_trees.md) - Hierarchical relationships
- [10. Heaps and Priority Queues](Heaps_and_Priority_Queues/00_heaps_and_priority_queues.md) - Priority-based access
- [11. Graphs](Graphs/00_graphs.md) - Network relationships

## Memory Layout Fundamentals

### Contiguous vs Node-Based

| Aspect | Contiguous (Array) | Node-Based (Linked) |
|--------|-------------------|---------------------|
| Memory Layout | Sequential addresses | Scattered with pointers |
| Cache Performance | Excellent (locality) | Poor (pointer chasing) |
| Random Access | O(1) | O(n) |
| Insertion/Deletion | O(n) shifting | O(1) with reference |
| Memory Overhead | Minimal | Pointer per element |

### Practical Implications

**Cache-friendly structures** (arrays, heaps) outperform node-based structures in practice due to CPU cache effects, even when Big-O suggests otherwise. A linked list traversal can be 10-100x slower than array traversal despite both being O(n).

## Selection Framework

```
Need O(1) random access?
├── Yes → Array or Hash Table
└── No → What's the access pattern?
    ├── LIFO → Stack
    ├── FIFO → Queue
    ├── Priority-based → Heap
    ├── Hierarchical → Tree
    └── Relationships → Graph
```

## Common Interview Trade-offs

| Scenario | Better Choice | Why |
|----------|--------------|-----|
| Frequent lookups | Hash Table | O(1) average |
| Sorted data needed | BST or sorted array | Maintains order |
| Priority scheduling | Heap | O(log n) for min/max |
| Undo operations | Stack | Natural LIFO |
| Level-order processing | Queue | Natural FIFO |
| Prefix matching | Trie | Optimized for strings |
