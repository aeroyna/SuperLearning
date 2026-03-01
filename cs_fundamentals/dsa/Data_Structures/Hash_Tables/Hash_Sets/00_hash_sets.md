# Hash Sets

A hash set is a collection of unique elements with O(1) average-case membership testing, insertion, and deletion. Internally, it's a hash map where only keys matter (values are typically null or boolean).

## Overview

Hash sets provide:
- **O(1) membership test**: `element in set`
- **O(1) insertion**: `set.add(element)`
- **O(1) deletion**: `set.remove(element)`
- **Uniqueness guarantee**: Duplicates automatically ignored

## Topics

- [6.1.1 Hash Set Implementation](01_hash_set.md)

## Internal Mechanics

A hash set is essentially a hash map mapping keys to a constant:

```python
class HashSet:
    def __init__(self):
        self._map = {}  # key -> True (or any constant)

    def add(self, val):
        self._map[val] = True

    def remove(self, val):
        self._map.pop(val, None)

    def contains(self, val):
        return val in self._map
```

**Python's set**: Actually optimized—stores only keys without values, saving memory.

## Memory Comparison

For storing n integers:
- **List**: ~8n bytes (just the integers)
- **Set**: ~50n bytes (hash table overhead: hash codes, pointers, load factor space)

**Trade-off**: Sets use 5-6x more memory but provide O(1) lookup vs O(n) for lists.

## Set Operations

### Mathematical Set Operations

```python
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}

# Union: elements in either set
a | b       # {1, 2, 3, 4, 5, 6}
a.union(b)

# Intersection: elements in both sets
a & b       # {3, 4}
a.intersection(b)

# Difference: elements in a but not b
a - b       # {1, 2}
a.difference(b)

# Symmetric difference: elements in exactly one set
a ^ b       # {1, 2, 5, 6}
a.symmetric_difference(b)
```

**Complexity of set operations**:
| Operation | Time Complexity |
|-----------|-----------------|
| Union | O(len(a) + len(b)) |
| Intersection | O(min(len(a), len(b))) |
| Difference | O(len(a)) |
| Symmetric difference | O(len(a) + len(b)) |

## Common Hash Set Patterns

### Pattern 1: Deduplication

>[!example]- C++
>```cpp
>vector<int> removeDuplicates(vector<int>& nums) {
>    unordered_set<int> uniqueElements(nums.begin(), nums.end());
>    return vector<int>(uniqueElements.begin(), uniqueElements.end());
>}
>```

>[!example]- Java
>```java
>public List<Integer> removeDuplicates(int[] nums) {
>    Set<Integer> uniqueElements = new HashSet<>();
>    for (int num : nums) uniqueElements.add(num);
>    return new ArrayList<>(uniqueElements);
>}
>```

>[!example]- Python
>```python
>def remove_duplicates(nums):
>    return list(set(nums))
>
># Order-preserving deduplication (Python 3.7+)
>def remove_duplicates_ordered(nums):
>    return list(dict.fromkeys(nums))
>```

>[!example]- JavaScript
>```javascript
>function removeDuplicates(nums) {
>    return Array.from(new Set(nums));
>}
>```

### Pattern 2: Fast Lookup / Visited Tracking

```python
def contains_duplicate(nums):
    seen = set()
    for num in nums:
        if num in seen:
            return True
        seen.add(num)
    return False
```

### Pattern 3: Finding Missing/Extra Elements

```python
def find_disappeared_numbers(nums):
    all_nums = set(range(1, len(nums) + 1))
    present = set(nums)
    return list(all_nums - present)

def single_number(nums):
    # Using XOR is more efficient, but set approach:
    seen_once = set()
    seen_twice = set()
    for num in nums:
        if num in seen_once:
            seen_once.remove(num)
            seen_twice.add(num)
        elif num not in seen_twice:
            seen_once.add(num)
    return seen_once.pop()
```

### Pattern 4: Graph Visited Tracking

```python
def num_islands(grid):
    if not grid:
        return 0

    visited = set()
    count = 0

    def dfs(r, c):
        if (r, c) in visited:
            return
        if r < 0 or r >= len(grid) or c < 0 or c >= len(grid[0]):
            return
        if grid[r][c] == '0':
            return

        visited.add((r, c))
        dfs(r + 1, c)
        dfs(r - 1, c)
        dfs(r, c + 1)
        dfs(r, c - 1)

    for i in range(len(grid)):
        for j in range(len(grid[0])):
            if grid[i][j] == '1' and (i, j) not in visited:
                dfs(i, j)
                count += 1

    return count
```

### Pattern 5: Longest Consecutive Sequence

```python
def longest_consecutive(nums):
    num_set = set(nums)
    max_length = 0

    for num in num_set:
        # Only start from sequence beginning
        if num - 1 not in num_set:
            current = num
            length = 1
            while current + 1 in num_set:
                current += 1
                length += 1
            max_length = max(max_length, length)

    return max_length
```

**Why check `num - 1 not in set`**: Ensures we only start counting from the beginning of each sequence, avoiding redundant work.

## frozenset: Immutable Hash Set

```python
# Regular set (mutable) - cannot be dict key or set element
regular = {1, 2, 3}

# Frozen set (immutable) - can be dict key or set element
frozen = frozenset([1, 2, 3])

# Use case: tracking visited states (like in BFS)
visited_states = set()
state = frozenset([1, 2, 3])  # Current positions
visited_states.add(state)
```

## Common Pitfalls

1. **Unhashable types**: Lists and dicts cannot be added to sets—use tuples or frozensets
2. **Set order**: Sets are unordered; don't rely on iteration order for logic
3. **Modifying during iteration**: Avoid `for x in s: s.remove(x)`—iterate over a copy
4. **Memory for small collections**: For < ~10 elements, a list might be more efficient

## Complexity Summary

| Operation | Average | Worst |
|-----------|---------|-------|
| add | O(1) | O(n) |
| remove | O(1) | O(n) |
| in | O(1) | O(n) |
| len | O(1) | O(1) |

## Key Interview Problems

| Problem | Pattern | Difficulty | LeetCode Link |
| --------- | --------- | ------------ | --- |
| Contains Duplicate | Membership | Easy | [Link](https://leetcode.com/problems/contains-duplicate/) |
| Intersection of Arrays | Set operations | Easy | [Link](https://leetcode.com/problems/intersection-of-two-arrays/) |
| Longest Consecutive Sequence | Sequence detection | Medium | [Link](https://leetcode.com/problems/longest-consecutive-sequence/) |
| Valid Sudoku | Row/col/box sets | Medium | [Link](https://leetcode.com/problems/valid-sudoku/) |
| Word Break | Membership + DP | Medium | [Link](https://leetcode.com/problems/word-break/) |
