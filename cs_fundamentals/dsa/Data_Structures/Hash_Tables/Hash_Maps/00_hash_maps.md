# Hash Maps

Hash maps (dictionaries) store key-value pairs with O(1) average-case operations for insertion, deletion, and lookup. Understanding the internal mechanics—hashing, collision resolution, and load factors—is essential for both using them effectively and answering interview questions about their behavior.

## Overview

A hash map uses a hash function to convert keys into array indices, providing constant-time access. The hash function must be:
- **Deterministic**: Same key always produces same hash
- **Uniform**: Distributes keys evenly across buckets
- **Fast**: O(1) to compute

## Topics

- [6.2.1 Hash Map Implementation](01_hash_map.md)

## Internal Architecture

### Memory Layout

```
Hash Table (array of buckets):
┌───────────────────────────────────────────────────────────┐
│ [0]  │ [1]  │ [2]  │ [3]  │ [4]  │ [5]  │ [6]  │ [7]   │
│  ↓   │  ↓   │  ↓   │  ↓   │  ↓   │  ↓   │  ↓   │  ↓    │
│ null │ K→V  │ K→V  │ null │ K→V  │ null │ K→V→V│ null  │
│      │      │      │      │      │      │ chain│       │
└───────────────────────────────────────────────────────────┘
```

### Hash Function Process

```python
def get_bucket_index(key, num_buckets):
    hash_code = hash(key)           # Python's built-in hash
    index = hash_code % num_buckets  # Map to valid index
    return index
```

**Collision example**: If `hash("apple") % 8 == 3` and `hash("banana") % 8 == 3`, both keys land in bucket 3.

## Collision Resolution Strategies

### Chaining (Most Common)

Each bucket contains a linked list of entries:

```python
class HashMapChaining:
    def __init__(self, capacity=16):
        self._buckets = [[] for _ in range(capacity)]
        self._size = 0

    def put(self, key, value):
        idx = hash(key) % len(self._buckets)
        for i, (k, v) in enumerate(self._buckets[idx]):
            if k == key:
                self._buckets[idx][i] = (key, value)  # Update
                return
        self._buckets[idx].append((key, value))  # Insert
        self._size += 1

    def get(self, key):
        idx = hash(key) % len(self._buckets)
        for k, v in self._buckets[idx]:
            if k == key:
                return v
        raise KeyError(key)
```

### Open Addressing (Linear Probing)

On collision, find next empty slot:

```python
class HashMapOpenAddressing:
    def put(self, key, value):
        idx = hash(key) % len(self._buckets)
        while self._buckets[idx] is not None:
            if self._buckets[idx][0] == key:
                self._buckets[idx] = (key, value)
                return
            idx = (idx + 1) % len(self._buckets)  # Linear probe
        self._buckets[idx] = (key, value)
```

**Trade-offs**:
| Aspect | Chaining | Open Addressing |
|--------|----------|-----------------|
| Cache locality | Poor (pointer chasing) | Good (sequential memory) |
| Memory | Extra for linked list nodes | In-place |
| Deletion | Simple (remove from list) | Complex (tombstones) |
| Load factor tolerance | > 1 allowed | Must be < 1 |

## Load Factor and Resizing

**Load factor** = n/k (items / buckets)

```python
def _maybe_resize(self):
    if self._size / len(self._buckets) > 0.75:  # Threshold
        self._resize(2 * len(self._buckets))

def _resize(self, new_capacity):
    old_buckets = self._buckets
    self._buckets = [[] for _ in range(new_capacity)]
    self._size = 0
    for bucket in old_buckets:
        for key, value in bucket:
            self.put(key, value)  # Rehash all entries
```

**Why rehash**: Bucket index depends on number of buckets. After resize, all keys may map to different buckets.

## Common Hash Map Patterns

### Two Sum Pattern

>[!example]- C++
>```cpp
>vector<int> twoSum(vector<int>& nums, int target) {
>    unordered_map<int, int> seen;
>    for (int i = 0; i < nums.size(); i++) {
>        int complement = target - nums[i];
>        if (seen.count(complement)) {
>            return {seen[complement], i};
>        }
>        seen[nums[i]] = i;
>    }
>    return {};
>}
>```

>[!example]- Java
>```java
>public int[] twoSum(int[] nums, int target) {
>    Map<Integer, Integer> seen = new HashMap<>();
>    for (int i = 0; i < nums.length; i++) {
>        int complement = target - nums[i];
>        if (seen.containsKey(complement)) {
>            return new int[]{seen.get(complement), i};
>        }
>        seen.put(nums[i], i);
>    }
>    return new int[0];
>}
>```

>[!example]- Python
>```python
>def two_sum(nums, target):
>    seen = {}  # value -> index
>    for i, num in enumerate(nums):
>        complement = target - num
>        if complement in seen:
>            return [seen[complement], i]
>        seen[num] = i
>    return []
>```

>[!example]- JavaScript
>```javascript
>function twoSum(nums, target) {
>    const seen = new Map();
>    for (let i = 0; i < nums.length; i++) {
>        const complement = target - nums[i];
>        if (seen.has(complement)) {
>            return [seen.get(complement), i];
>        }
>        seen.set(nums[i], i);
>    }
>    return [];
>}
>```

### Counting Frequency

```python
from collections import Counter

def top_k_frequent(nums, k):
    count = Counter(nums)
    return [item for item, _ in count.most_common(k)]
```

### Grouping by Key

```python
from collections import defaultdict

def group_anagrams(strs):
    groups = defaultdict(list)
    for s in strs:
        key = tuple(sorted(s))  # Or: tuple of char counts
        groups[key].append(s)
    return list(groups.values())
```

## Complexity Analysis

| Operation | Average | Worst Case | Notes |
|-----------|---------|------------|-------|
| Get | O(1) | O(n) | All keys collide |
| Put | O(1)* | O(n) | *Amortized (resize) |
| Delete | O(1) | O(n) | All keys collide |
| Iteration | O(n) | O(n) | Visit all entries |

**When worst case occurs**: Poor hash function or adversarial input causing all keys to collide.

## Common Pitfalls

1. **Mutable keys**: Never use mutable objects (lists, dicts) as keys
2. **Custom objects**: Must implement `__hash__` and `__eq__` consistently
3. **Iteration order**: In Python 3.7+, dicts maintain insertion order (but don't rely on this conceptually)
4. **Integer overflow**: In some languages, hash codes can overflow
5. **Default values**: Use `dict.get(key, default)` or `defaultdict` to avoid KeyError

## Key Interview Problems

| Problem | Pattern | Difficulty | LeetCode Link |
| --------- | --------- | ------------ | --- |
| Two Sum | Complement lookup | Easy | [Link](https://leetcode.com/problems/two-sum/) |
| Group Anagrams | Grouping by key | Medium | [Link](https://leetcode.com/problems/group-anagrams/) |
| Top K Frequent Elements | Counting | Medium | [Link](https://leetcode.com/problems/top-k-frequent-elements/) |
| Longest Consecutive Sequence | Set membership | Medium | [Link](https://leetcode.com/problems/longest-consecutive-sequence/) |
| LRU Cache | Hash map + linked list | Medium | [Link](https://leetcode.com/problems/lru-cache/) |
| Design HashMap | Implementation | Medium | [Link](https://leetcode.com/problems/design-hashmap/) |
