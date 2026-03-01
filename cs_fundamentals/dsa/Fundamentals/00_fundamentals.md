# Fundamentals

The foundation of algorithmic thinking rests on understanding how to measure efficiency and select appropriate data structures. Before diving into specific implementations, mastering these fundamentals ensures you can reason about trade-offs, predict performance, and communicate solutions effectively in interviews.

## Overview

This section establishes the analytical framework used throughout DSA study:

1. **Complexity Analysis** - The language we use to describe algorithm efficiency
2. **Data Structure Selection** - Understanding the relationship between abstract concepts and concrete implementations

## Topics

### [1. Big-O Complexity](Big_O_Complexity/00_big_o_complexity.md)

The mathematical framework for analyzing algorithm efficiency. Covers asymptotic notation, amortized analysis, and the practical mapping between problem constraints and expected time complexity.

- [1.1 Time and Space Complexity](Big_O_Complexity/01_time_and_space_complexity.md)
- [1.2 Input Size Guidelines](Big_O_Complexity/02_input_size_guidelines.md)
- [1.3 Complexity Cheatsheet](Big_O_Complexity/03_complexity_cheatsheet.md)

### [2. Abstract vs Concrete Data Structures](Abstract_vs_Concrete_DS/00_abstract_vs_concrete_ds.md)

The distinction between what a data structure does (its interface) versus how it's implemented (its mechanics). This conceptual separation is crucial for selecting optimal implementations.

- [2.1 Abstract Data Types](Abstract_vs_Concrete_DS/01_abstract_data_types.md)
- [2.2 Choosing the Right Structure](Abstract_vs_Concrete_DS/02_choosing_the_right_structure.md)

## Why This Matters

### Interview Context

Every coding interview implicitly tests your understanding of these fundamentals:

- **"What's the time complexity?"** - Requires Big-O fluency
- **"Can you optimize this?"** - Requires understanding complexity classes
- **"Why did you choose this data structure?"** - Requires understanding ADT vs implementation trade-offs

### Mental Model

Think of fundamentals as the "grammar" of DSA:

```
Problem Constraints → Expected Complexity → Viable Data Structures → Algorithm Design
```

For example:
- n ≤ 10^5 with time limit → O(n log n) acceptable → Sorting, heap, or binary search viable
- Need O(1) lookup → Hash table implementation, not tree-based

## Common Pitfalls

1. **Ignoring Space Complexity** - Time is discussed more, but space constraints matter equally
2. **Best Case Fixation** - Interviewers care about worst and average case
3. **Confusing ADT with Implementation** - A "stack" can be array-based or linked-list-based
4. **Forgetting Amortization** - Dynamic array append is O(1) amortized, not worst-case
