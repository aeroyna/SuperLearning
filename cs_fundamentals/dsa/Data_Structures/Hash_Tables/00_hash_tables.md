# Hash Tables

Hash tables (hash maps/dictionaries) provide O(1) average-case lookup, insert, and delete operations. They are one of the most frequently used data structures in coding interviews.

## Overview

A hash table stores key-value pairs using a hash function to compute an index into an array of buckets.

## Topics

- [6.1 Hash Set](Hash_Sets/01_hash_set.md)
- [6.2 Hash Map](Hash_Maps/01_hash_map.md)
- [6.3 Designing Hash Keys](02_designing_hash_keys.md)
- [6.4 Counting with Hash Maps](03_counting_with_hash_maps.md)
- [6.5 Practice Problems](Practice_Problems/00_practice_problems.md)

## Key Operations

| Operation | Average | Worst |
|-----------|---------|-------|
| Search | O(1) | O(n) |
| Insert | O(1) | O(n) |
| Delete | O(1) | O(n) |

*Worst case occurs with many collisions*

## Hash Set vs Hash Map

| Hash Set | Hash Map |
|----------|----------|
| Stores unique keys only | Stores key-value pairs |
| Check existence | Store associated data |
| `{1, 2, 3}` | `{1: "a", 2: "b"}` |

## Common Use Cases

### 1. Existence Check (Hash Set)

>[!example]- C++
>```cpp
>unordered_set<int> seen;
>for (int num : nums) {
>    if (seen.count(num)) return true; // Duplicate found
>    seen.insert(num);
>}
>```

>[!example]- Java
>```java
>Set<Integer> seen = new HashSet<>();
>for (int num : nums) {
>    if (seen.contains(num)) return true; // Duplicate found
>    seen.add(num);
>}
>```

>[!example]- Python
>```python
>seen = set()
>for num in nums:
>    if num in seen:
>        return True  # Duplicate found
>    seen.add(num)
>```

>[!example]- JavaScript
>```javascript
>const seen = new Set();
>for (const num of nums) {
>    if (seen.has(num)) return true; // Duplicate found
>    seen.add(num);
>}
>```

### 2. Counting Frequency (Hash Map)

>[!example]- C++
>```cpp
>unordered_map<int, int> count;
>for (int num : nums) {
>    count[num]++;
>}
>```

>[!example]- Java
>```java
>Map<Integer, Integer> count = new HashMap<>();
>for (int num : nums) {
>    count.put(num, count.getOrDefault(num, 0) + 1);
>}
>```

>[!example]- Python
>```python
>from collections import Counter
>count = Counter(nums)
># or
>count = {}
>for num in nums:
>    count[num] = count.get(num, 0) + 1
>```

>[!example]- JavaScript
>```javascript
>const count = new Map();
>for (const num of nums) {
>    count.set(num, (count.get(num) || 0) + 1);
>}
>```

### 3. Two Sum Pattern

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
>            return new int[] { seen.get(complement), i };
>        }
>        seen.put(nums[i], i);
>    }
>    return new int[0];
>}
>```

>[!example]- Python
>```python
>def twoSum(nums, target):
>    seen = {}
>    for i, num in enumerate(nums):
>        complement = target - num
>        if complement in seen:
>            return [seen[complement], i]
>        seen[num] = i
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

### 4. Grouping by Key

>[!example]- C++
>```cpp
>unordered_map<string, vector<string>> groups;
>for (const auto& item : items) {
>    string key = compute_key(item);
>    groups[key].push_back(item);
>}
>```

>[!example]- Java
>```java
>Map<String, List<String>> groups = new HashMap<>();
>for (String item : items) {
>    String key = computeKey(item);
>    groups.computeIfAbsent(key, k -> new ArrayList<>()).add(item);
>}
>```

>[!example]- Python
>```python
>from collections import defaultdict
>groups = defaultdict(list)
>for item in items:
>    key = compute_key(item)
>    groups[key].append(item)
>```

>[!example]- JavaScript
>```javascript
>const groups = new Map();
>for (const item of items) {
>    const key = computeKey(item);
>    if (!groups.has(key)) groups.set(key, []);
>    groups.get(key).push(item);
>}
>```

## Interview Patterns

| Pattern | Description | Example |
|---------|-------------|---------|
| Two Sum | Find pair with property | Two Sum, 3Sum |
| Counting | Frequency analysis | Top K Frequent |
| Grouping | Group by computed key | Group Anagrams |
| Caching | Memoization | LRU Cache |
| Sliding Window | Track window state | Longest Substring |
| Subarray Sum | Prefix sum + hash | Subarray Sum = K |

## Common Mistakes

1. **Mutable keys**: Don't use lists as dictionary keys
2. **Missing key access**: Use `.get()` or `defaultdict`
3. **Hash collisions**: Know they exist, usually not your problem
4. **Order assumption**: Regular dicts are ordered in Python 3.7+
