# Bitmask DP

Bitmask DP is an advanced Dynamic Programming technique used when the state of a problem needs to represent a subset of items, and the number of items is small (typically `N <= 20`).

## Concept

Instead of using a `Set` or `Boolean Array` in our DP state (which is slow to hash or copy), we use an **integer** to represent the set.
- If the `i`-th bit is `1`, the item `i` is included/visited.
- If the `i`-th bit is `0`, the item `i` is excluded/unvisited.

## Key Patterns

1.  **[Traveling Salesperson (TSP)](01_traveling_salesperson.md)**:
    - Finding shortest paths that visit every node exactly once.
    - State: `dp[mask][last_visited]`.

2.  **[Subsets and Partitioning](02_subsets_and_profiles.md)**:
    - Partitioning a set into groups or matching problems.
    - State: `dp[mask]` (represents "is this subset valid" or "current progress").

## Pattern Recognition

**Signals**:
- "Find shortest path visiting all nodes" (TSP)
- "Assign N people to N tasks to minimize cost"
- "Small constraints", e.g., `N <= 20`
- "Subsets", "Partitioning sets"

## Common Problems

### 1. Smallest Sufficient Team
Find smallest team to cover all required skills.
- **State**: `dp[skill_mask]` = min people count.
- **Transition**: `dp[mask | person_skills] = min(...)`

### 2. Maximum Students Taking Exam
Seats arrangement constraints.
- **State**: `dp[row][mask]` where mask represents occupied seats in current row.

### Practice
- [Practice Problems](Practice_Problems/00_practice_problems.md)