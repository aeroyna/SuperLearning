# Knapsack Pattern

The Knapsack problem is a fundamental Dynamic Programming pattern. It involves selecting a subset of items with given weights and values to maximize the total value while staying within a weight limit (capacity).

## Overview

We classify Knapsack problems based on item constraints:

1.  **[0/1 Knapsack](01_0_1_knapsack.md)**: Each item can be picked at most **once**.
2.  **[Unbounded Knapsack](02_unbounded_knapsack.md)**: Each item can be picked **unlimited** times.

## Pattern Recognition

**Signals**:
- "Select items to maximize value with capacity constraint"
- "Partition array into two subsets with equal sum"
- "Count ways to reach target sum using numbers"
- "Minimum coins to make change"

## Common Variations

### 1. Partition Equal Subset Sum
Can array be partitioned into two subsets with equal sum?
- **Mapping**: Capacity = TotalSum / 2. Item values = weights = nums[i].
- **Type**: 0/1 Knapsack.

### 2. Target Sum
Assign `+` or `-` to nums to match target `S`.
- **Mapping**: Reduces to finding a subset P such that `sum(P) = (S + TotalSum) / 2`.
- **Type**: 0/1 Knapsack.

### 3. Coin Change II
Number of ways to make amount using coins.
- **Mapping**: Weights = coin values. Capacity = amount.
- **Type**: Unbounded Knapsack.

### Practice
- [Practice Problems](Practice_Problems/00_practice_problems.md)