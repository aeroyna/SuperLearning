# Islands (Matrix Traversal)

The "Islands" pattern is a fundamental class of problems involving grid (matrix) traversal. It is essentially graph theory applied to 2D arrays.

## Overview

Most island problems can be solved using either **DFS** (Depth-First Search) or **BFS** (Breadth-First Search).

### Core Concepts
1.  **[Number of Islands (DFS)](01_number_of_islands.md)**: The foundational pattern. Visiting connected components.
2.  **[Matrix BFS (Shortest Path)](02_matrix_bfs.md)**: Finding shortest paths or simulating spread (Rotting Oranges).

## Pattern Recognition

**Signals**:
- "Grid", "Matrix", "Connected components"
- "Islands", "Regions", "Enclosed areas"
- "Shortest path in a grid" (BFS)
- "Reach destination"

## Common Problems

### 1. Max Area of Island
Find the area of the largest island.
- **Approach**: DFS. Modify `dfs` to return `1 + sum(neighbors)`.

### 2. Rotting Oranges
Minimum time for all fresh oranges to rot.
- **Approach**: Multi-source BFS.

### 3. Word Search
Exist word in grid?
- **Approach**: Backtracking DFS.

### Practice
- [Practice Problems](Practice_Problems/00_practice_problems.md)