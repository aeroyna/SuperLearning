# Choosing the Right Data Structure

A systematic approach to selecting the appropriate data structure for any problem.

## Decision Framework

### Step 1: Identify Required Operations

Ask yourself:
- What data do I need to store?
- What operations do I need? (insert, delete, search, access, iterate)
- How frequent is each operation?
- What are the time complexity requirements?

### Step 2: Consider Constraints

- Memory limitations?
- Need for ordering?
- Duplicate elements allowed?
- Random access required?

### Step 3: Match to Data Structure

## Quick Selection Guide

### Need O(1) Random Access by Position?

→ **Use Array/ArrayList**
- Direct indexing
- Cache-friendly
- Best for: Fixed-size collections, frequent access by index

### Need O(1) Lookup by Key/Value?

→ **Use Hash Map/Hash Set**
- Constant time average operations
- Best for: Counting, deduplication, lookups

### Need Sorted Order with Fast Operations?

→ **Use Balanced BST (TreeMap/TreeSet)**
- O(log n) insert, delete, search
- Maintains sorted order
- Best for: Range queries, floor/ceiling operations

### Need Frequent Insert/Delete at Both Ends?

→ **Use Deque**
- O(1) operations at both ends
- Best for: Sliding window, BFS

### Need Frequent Insert/Delete in Middle?

→ **Use Linked List**
- O(1) insert/delete with pointer
- O(n) access
- Best for: When modifications are more frequent than access

### Need Min/Max Repeatedly?

→ **Use Heap/Priority Queue**
- O(1) get min/max
- O(log n) insert/extract
- Best for: Top-K problems, scheduling, merging sorted lists

### Need LIFO Access Pattern?

→ **Use Stack**
- Best for: Parsing, backtracking, undo operations

### Need FIFO Access Pattern?

→ **Use Queue**
- Best for: BFS, level-order traversal, buffering

### Need Prefix-Based Operations on Strings?

→ **Use Trie**
- Best for: Autocomplete, spell check, prefix matching

### Need Range Updates/Queries?

→ **Use Segment Tree or Fenwick Tree**
- O(log n) range operations
- Best for: Competitive programming, range sum queries

## Common Problem Type → Data Structure

| Problem Type               | Primary Data Structure        | Alternative                 |
| -------------------------- | ----------------------------- | --------------------------- |
| Two Sum variants           | Hash Map                      | Sorted Array + Two Pointers |
| Sliding Window             | Deque, Hash Map               | Two Pointers                |
| Top-K elements             | Heap                          | QuickSelect                 |
| Graph traversal            | Adjacency List + Queue/Stack  | Adjacency Matrix            |
| Shortest path (weighted)   | Adjacency List + Heap         | -                           |
| Shortest path (unweighted) | Adjacency List + Queue        | -                           |
| Level-order traversal      | Queue                         | -                           |
| DFS traversal              | Stack (or recursion)          | -                           |
| Cycle detection            | Set + DFS/BFS                 | Union-Find                  |
| Connected components       | Union-Find                    | DFS/BFS                     |
| Expression evaluation      | Stack                         | -                           |
| Parentheses matching       | Stack                         | -                           |
| Merge K sorted lists       | Min-Heap                      | -                           |
| Find median                | Two Heaps                     | -                           |
| LRU Cache                  | Hash Map + Doubly Linked List | -                           |
| Interval problems          | Arrays + Sorting              | Interval Tree               |

## Trade-off Analysis

### Array vs Linked List

| Operation | Array | Linked List |
|-----------|-------|-------------|
| Access by index | O(1) | O(n) |
| Insert at end | O(1)* | O(1) |
| Insert at beginning | O(n) | O(1) |
| Insert in middle | O(n) | O(1)** |
| Memory overhead | Low | High (pointers) |
| Cache performance | Excellent | Poor |

*Amortized for dynamic arrays
**After reaching the position

### HashMap vs TreeMap

| Aspect | HashMap | TreeMap |
|--------|---------|---------|
| Get/Put | O(1) avg | O(log n) |
| Ordering | None | Sorted |
| Range queries | Not efficient | O(log n + k) |
| Memory | Higher | Lower |
| Use when | Need speed | Need order |

### HashSet vs TreeSet

Same trade-offs as HashMap vs TreeMap, but for unique elements without values.

### ArrayList vs LinkedList

**ArrayList** wins for:
- Random access
- Iteration
- Memory efficiency
- Cache friendliness

**LinkedList** wins for:
- Frequent insertion/deletion at known positions
- Queue operations (use ArrayDeque instead in practice)

## Real Interview Scenarios

### Scenario 1: "Implement a cache with O(1) get and put"

**Analysis**:
- Need fast lookup → Hash Map
- Need to track access order → Linked List
- **Solution**: Hash Map + Doubly Linked List (LRU Cache)

### Scenario 2: "Find running median in a stream"

**Analysis**:
- Need to maintain sorted order
- Need middle element quickly
- **Solution**: Two Heaps (max-heap for lower half, min-heap for upper half)

### Scenario 3: "Check if two strings are anagrams"

**Analysis**:
- Need to count character frequencies
- Need to compare counts
- **Solution**: Hash Map (or fixed-size array for ASCII)

### Scenario 4: "Find shortest path in unweighted graph"

**Analysis**:
- Unweighted → All edges equal
- Need to explore by distance
- **Solution**: BFS with Queue
