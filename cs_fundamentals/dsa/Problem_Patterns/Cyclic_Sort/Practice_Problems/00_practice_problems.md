# Practice Problems: Cyclic Sort

Sorting 1 to N in O(N).

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Missing Number](https://leetcode.com/problems/missing-number/) | Easy | Sort `nums[i]` to index `nums[i]`. Find index mismatch. |
| [Find the Duplicate Number](https://leetcode.com/problems/find-the-duplicate-number/) | Medium | If `nums[i]` belongs at `nums[i]-1` and it's occupied by same val, duplicate. |
| [First Missing Positive](https://leetcode.com/problems/first-missing-positive/) | Hard | Ignore non-positives. Place `x` at `x-1`. First index `i` where `nums[i] != i+1`. |
