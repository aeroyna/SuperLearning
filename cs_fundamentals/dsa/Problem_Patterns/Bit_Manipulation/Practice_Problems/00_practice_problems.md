# Practice Problems: Bit Manipulation

Bitwise operations and optimizations.

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Single Number](https://leetcode.com/problems/single-number/) | Easy | XOR all numbers. `x ^ x = 0`. |
| [Counting Bits](https://leetcode.com/problems/counting-bits/) | Easy | `dp[i] = dp[i >> 1] + (i & 1)`. |
| [Reverse Bits](https://leetcode.com/problems/reverse-bits/) | Easy | Iterate and shift bits. |
| [Sum of Two Integers](https://leetcode.com/problems/sum-of-two-integers/) | Medium | `(a ^ b)` is sum, `(a & b) << 1` is carry. |
| [Subsets](https://leetcode.com/problems/subsets/) | Medium | Iterate `0` to `2^n - 1`. Mask determines subset. |