# Input Size Guidelines

Understanding the relationship between input constraints and expected time complexity is crucial for interviews. This guide helps you determine the appropriate algorithm based on problem constraints.

## Constraint to Complexity Mapping

### n <= 10

**Expected Complexity**: O(n! * n) or O(n^2 * n!) or O(4^n)

**Approach**:
- Brute force is acceptable
- Backtracking with heavy computation
- Generate all permutations/combinations

**Common Problems**:
- N-Queens (n <= 9)
- Traveling Salesman (small n)
- Problems requiring all permutations

---

### 10 < n <= 20

**Expected Complexity**: O(2^n) or O(2^n * n)

**Approach**:
- Subsets/subsequences (each element: take or don't take)
- Bitmask DP
- Meet-in-the-middle technique

**Common Problems**:
- Subset Sum
- Partition problems
- Bitmask enumeration

> Note: 2^20 ≈ 1 million, which is manageable

---

### 20 < n <= 100

**Expected Complexity**: O(n^3) or O(n^4)

**Approach**:
- Triple/quadruple nested loops
- Floyd-Warshall (all-pairs shortest path)
- Some matrix operations

**Common Problems**:
- Matrix chain multiplication
- Interval DP
- Some graph problems

---

### 100 < n <= 1,000

**Expected Complexity**: O(n^2) or O(n^2 log n)

**Approach**:
- Double nested loops
- 2D DP
- Some sorting-based solutions

**Common Problems**:
- Longest Common Subsequence
- Edit Distance
- Most "medium" DP problems

---

### 1,000 < n <= 100,000 (10^5)

**Expected Complexity**: O(n log n) or O(n)

**This is the most common constraint on LeetCode!**

**Approach**:
- Sorting + linear scan
- Binary search
- Two pointers
- Sliding window
- Hash maps
- Monotonic stack
- Heap operations
- Single-pass DP

**Common Problems**:
- Most array/string problems
- Graph BFS/DFS
- Tree problems

**What WON'T work**: O(n^2) nested loops

---

### 100,000 < n <= 1,000,000 (10^6)

**Expected Complexity**: O(n) strongly preferred, O(n log n) with small constant

**Approach**:
- Single pass algorithms
- Hash map solutions
- Prefix sums
- Two pointers

**What WON'T work**: Any algorithm with more than O(n log n)

---

### n > 1,000,000 (10^6+)

**Expected Complexity**: O(log n) or O(1)

**Approach**:
- Binary search on answer
- Mathematical formulas
- Clever use of hash maps
- Bit manipulation

**Common Problems**:
- Finding nth Fibonacci (matrix exponentiation)
- Binary search on answer space
- Mathematical problems

---

## Quick Reference Table

| n | Max Complexity | Common Approaches |
|---|----------------|-------------------|
| ≤ 10 | O(n!) | Backtracking, brute force |
| ≤ 20 | O(2^n) | Subsets, bitmask |
| ≤ 100 | O(n^3) | Triple loops, Floyd-Warshall |
| ≤ 1,000 | O(n^2) | Nested loops, 2D DP |
| ≤ 10^5 | O(n log n) | Sort + scan, binary search, heap |
| ≤ 10^6 | O(n) | Hash map, two pointers, prefix sum |
| > 10^6 | O(log n) | Binary search, math |

## Operations Per Second Rule of Thumb

Modern computers perform roughly **10^8 to 10^9** simple operations per second.

| Operations | Time |
|------------|------|
| 10^6 | ~1ms |
| 10^7 | ~10ms |
| 10^8 | ~100ms |
| 10^9 | ~1s (usually TLE) |

## Interview Tips

1. **Always ask about constraints** if not given
2. **Start with brute force** to understand the problem
3. **Calculate complexity** before coding
4. **If TLE is likely**, optimize before implementing

### Example Analysis

```
Problem: Find all pairs (i, j) where arr[i] + arr[j] = target
Constraint: n = 10^5

Brute force: O(n^2) = 10^10 operations = TLE!
Hash map solution: O(n) = 10^5 operations = Fast!
```

## Special Cases

### Constant Factors Matter

Sometimes O(n) with a large constant can be slower than O(n log n):
- O(26n) for alphabet iteration
- O(n * 32) for bit manipulation

These are still technically O(n) but be aware of hidden constants.

### Multiple Variables

When problem has multiple inputs:
- `n` items, `k` operations: Consider both in complexity
- Two arrays of size `n` and `m`: O(n + m) or O(n * m)?
- Graph with V vertices, E edges: O(V + E)
