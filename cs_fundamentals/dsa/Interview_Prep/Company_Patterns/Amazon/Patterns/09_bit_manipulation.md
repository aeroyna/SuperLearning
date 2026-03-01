# Amazon Bit Manipulation Patterns

**Frequency**: 🟡 **Low/Medium**

Bit manipulation problems are less frequent but can appear in Amazon interviews, often for optimizing space or performing fast calculations. They test a deeper understanding of integer representation.

## Key Concepts
- **Bitwise Operators**: AND (&), OR (|), XOR (^), NOT (~), Left Shift (<<), Right Shift (>>).
- **Bitmasks**: Using integers to represent sets of flags or subsets.
- **Bit Hacking**: Clever tricks for common operations (e.g., counting set bits, checking power of 2).

## Phase 1: Must-Do (Foundation)

Master these 10 problems to build a solid foundation.

| Problem | Difficulty | Key Concept |
| :--- | :--- | :--- |
| [Single Number](https://leetcode.com/problems/single-number/) | Easy | XOR property: `a ^ a = 0`. |
| [Number of 1 Bits](https://leetcode.com/problems/number-of-1-bits/) | Easy | Brian Kernighan's algorithm. |
| [Counting Bits](https://leetcode.com/problems/counting-bits/) | Easy | DP: `count[i] = count[i >> 1] + (i & 1)`. |
| [Reverse Bits](https://leetcode.com/problems/reverse-bits/) | Easy | Iterate and shift. |
| [Sum of Two Integers](https://leetcode.com/problems/sum-of-two-integers/) | Medium | Adder logic without `+` operator. |
| [Missing Number](https://leetcode.com/problems/missing-number/) | Easy | XOR with range or Summation. |
| [Power of Two](https://leetcode.com/problems/power-of-two/) | Easy | `n > 0` and `(n & (n - 1)) == 0`. |
| [Find the Difference](https://leetcode.com/problems/find-the-difference/) | Easy | XOR or character sum. |
| [Bitwise AND of Numbers Range](https://leetcode.com/problems/bitwise-and-of-numbers-range/) | Medium | Find common prefix. |
| [Subsets](https://leetcode.com/problems/subsets/) | Medium | Bitmasking to generate all subsets. |

## Phase 2: Practice & Variants (Depth)

Tackle these 10 harder variations and common follow-ups.

| Problem | Difficulty | Key Concept |
| :--- | :--- | :--- |
| [Divide Two Integers](https://leetcode.com/problems/divide-two-integers/) | Medium | Bit manipulation (Shift/Subtract). |
| [Total Hamming Distance](https://leetcode.com/problems/total-hamming-distance/) | Medium | Count set bits at each position. |
| [UTF-8 Validation](https://leetcode.com/problems/utf-8-validation/) | Medium | Bitwise checks for UTF-8 rules. |
| [Number of Matching Subsequences](https://leetcode.com/problems/number-of-matching-subsequences/) | Medium | Next character pointers. |
| [Repeated DNA Sequences](https://leetcode.com/problems/repeated-dna-sequences/) | Medium | Rolling hash / Bitmasking. |
| [Maximum XOR of Two Numbers in an Array](https://leetcode.com/problems/maximum-xor-of-two-numbers-in-an-array/) | Medium | Trie for XOR maximization. |
| [Hamming Distance](https://leetcode.com/problems/hamming-distance/) | Easy | XOR. |
| [Single Number III](https://leetcode.com/problems/single-number-iii/) | Medium | XOR + Rightmost set bit. |
| [Find the Kth Bit in Nth Binary String](https://leetcode.com/problems/find-the-kth-bit-in-nth-binary-string/) | Medium | Recursion + Bitwise patterns. |
| [Sum of Two Integers (no + or -)](https://leetcode.com/problems/sum-of-two-integers/) | Medium | Bitwise adder logic. |
