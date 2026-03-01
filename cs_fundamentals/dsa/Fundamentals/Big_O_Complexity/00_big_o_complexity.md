# Big-O Complexity

Understanding time and space complexity is fundamental to solving algorithmic problems efficiently. This section covers everything you need to know about Big-O notation for interviews.

## Overview

Big-O notation describes the upper bound of an algorithm's time or space requirements as the input size grows. It helps us:
- Compare algorithm efficiency
- Predict performance at scale
- Choose appropriate data structures
- Validate solutions against constraints

## Topics

- [1.1 Time and Space Complexity](01_time_and_space_complexity.md) - Understanding asymptotic analysis
- [1.2 Input Size Guidelines](02_input_size_guidelines.md) - Mapping constraints to expected complexity
- [1.3 Complexity Cheatsheet](03_complexity_cheatsheet.md) - Quick reference for all DS/A operations

## Key Concepts

### Common Complexities (Best to Worst)

| Complexity | Name | Example |
|------------|------|---------|
| O(1) | Constant | Array access, hash lookup |
| O(log n) | Logarithmic | Binary search |
| O(n) | Linear | Array traversal |
| O(n log n) | Linearithmic | Merge sort, heap sort |
| O(n^2) | Quadratic | Nested loops |
| O(2^n) | Exponential | Subsets generation |
| O(n!) | Factorial | Permutations |

### Quick Rules

1. **Drop constants**: O(2n) = O(n)
2. **Drop non-dominant terms**: O(n^2 + n) = O(n^2)
3. **Different inputs = different variables**: O(n + m), not O(n)
4. **Nested operations multiply**: Two nested loops = O(n * m)
5. **Sequential operations add**: Two separate loops = O(n + m)
