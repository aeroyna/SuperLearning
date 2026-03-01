# Practice Problems: Graphs

BFS, DFS, Union-Find, Topological Sort, and Shortest Paths.

## Traversal (BFS/DFS)

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Number of Islands](https://leetcode.com/problems/number-of-islands/) | Medium | DFS/BFS to visit connected `1`s. Mark visited. |
| [Clone Graph](https://leetcode.com/problems/clone-graph/) | Medium | DFS/BFS + Hash Map `old_node -> new_node`. |
| [Max Area of Island](https://leetcode.com/problems/max-area-of-island/) | Medium | DFS returning size of component. |

## Topological Sort

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Course Schedule](https://leetcode.com/problems/course-schedule/) | Medium | Cycle detection (DFS colors) or Kahn's Algo (in-degree). |
| [Course Schedule II](https://leetcode.com/problems/course-schedule-ii/) | Medium | Return order from Topological Sort. |
| [Alien Dictionary](https://leetcode.com/problems/alien-dictionary/) | Hard | Build graph from adjacent word chars, then Top Sort. |

## Shortest Path & Union Find

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Network Delay Time](https://leetcode.com/problems/network-delay-time/) | Medium | Dijkstra's algorithm. |
| [Graph Valid Tree](https://leetcode.com/problems/graph-valid-tree/) | Medium | Union-Find to check 1 component and no cycles (n-1 edges). |
| [Number of Connected Components](https://leetcode.com/problems/number-of-connected-components-in-an-undirected-graph/) | Medium | Union-Find or DFS count. |
