# Practice Problems: Hash Tables

Efficient lookups, counting, and tracking visited states.

## Hash Map

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Two Sum](https://leetcode.com/problems/two-sum/) | Easy | `target - num` in map? Map stores `value -> index`. |
| [Group Anagrams](https://leetcode.com/problems/group-anagrams/) | Medium | Key is sorted string or char count tuple. |
| [LRU Cache](https://leetcode.com/problems/lru-cache/) | Medium | Doubly Linked List + Hash Map (`key -> node`). |

## Hash Set

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Longest Consecutive Sequence](https://leetcode.com/problems/longest-consecutive-sequence/) | Medium | Put all in set. Only start counting if `num-1` not in set. |
| [Contains Duplicate](https://leetcode.com/problems/contains-duplicate/) | Easy | `len(set(nums)) < len(nums)`. |
| [Happy Number](https://leetcode.com/problems/happy-number/) | Easy | Detect cycle in sum of squares sequence using set. |
