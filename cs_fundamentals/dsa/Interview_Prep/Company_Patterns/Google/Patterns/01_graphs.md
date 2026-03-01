# Google Graph Patterns (BFS / DFS / Union-Find)

**Frequency**: 🔴 **Very High**

Google interviews have seen a significant surge in graph-related problems. The focus is often on grid-based traversals, connected components, and topological sorting.

## Key Concepts
- **Grid as Graph**: Treating a 2D matrix as a graph where cells are nodes and adjacent cells are edges.
- **Multi-Source BFS**: Starting BFS from multiple nodes simultaneously to simulate parallel expansion (e.g., Rotting Oranges).
- **Union-Find**: Efficiently managing disjoint sets, crucial for component counting and connectivity problems.
- **Topological Sort**: Ordering tasks with dependencies (Course Schedule).

## Phase 1: Must-Do (Foundation)

Master these 10 problems to build a solid foundation in Google-style graph questions.

| Problem | Difficulty | Key Concept |
| :--- | :--- | :--- |
| [Number of Islands](https://leetcode.com/problems/number-of-islands/) | Medium | Classic DFS/BFS to find connected components. |
| [Max Area of Island](https://leetcode.com/problems/max-area-of-island/) | Medium | DFS returning size of component. |
| [Rotting Oranges](https://leetcode.com/problems/rotting-oranges/) | Medium | Multi-source BFS for simultaneous spread. |
| [Clone Graph](https://leetcode.com/problems/clone-graph/) | Medium | DFS/BFS with Hash Map to handle cycles. |
| [Course Schedule](https://leetcode.com/problems/course-schedule/) | Medium | Cycle detection (DFS/Topological Sort). |
| [Course Schedule II](https://leetcode.com/problems/course-schedule-ii/) | Medium | Topological Sort order (Kahn's Algo). |
| [Graph Valid Tree](https://leetcode.com/problems/graph-valid-tree/) | Medium | Union-Find or DFS (check 1 component & no cycles). |
| [Number of Connected Components in an Undirected Graph](https://leetcode.com/problems/number-of-connected-components-in-an-undirected-graph/) | Medium | Union-Find or DFS. |
| [Word Ladder](https://leetcode.com/problems/word-ladder/) | Hard | BFS for shortest path in unweighted graph. |
| [Surrounded Regions](https://leetcode.com/problems/surrounded-regions/) | Medium | DFS from boundary to mark safe regions. |

## Phase 2: Practice & Variants (Depth)

Tackle these 10 harder variations to handle edge cases and advanced constraints.

| Problem | Difficulty | Key Concept |
| :--- | :--- | :--- |
| [Alien Dictionary](https://leetcode.com/problems/alien-dictionary/) | Hard | Topological Sort on character dependencies. |
| [Evaluate Division](https://leetcode.com/problems/evaluate-division/) | Medium | DFS/BFS on weighted graph (path product). |
| [Most Stones Removed with Same Row or Column](https://leetcode.com/problems/most-stones-removed-with-same-row-or-column/) | Medium | Union-Find (Row/Col as connected nodes). |
| [Longest Increasing Path in a Matrix](https://leetcode.com/problems/longest-increasing-path-in-a-matrix/) | Hard | DFS + Memoization (DP on Graph). |
| [Reconstruct Itinerary](https://leetcode.com/problems/reconstruct-itinerary/) | Hard | Eulerian Path (Hierholzer's) / DFS Backtracking. |
| [Making A Large Island](https://leetcode.com/problems/making-a-large-island/) | Hard | DFS (Component size) + Enumeration of 0s. |
| [Cracking the Safe](https://leetcode.com/problems/cracking-the-safe/) | Hard | Eulerian Path on de Bruijn graph. |
| [Bus Routes](https://leetcode.com/problems/bus-routes/) | Hard | BFS (Level = number of buses, not stops). |
| [Shortest Path in Binary Matrix](https://leetcode.com/problems/shortest-path-in-binary-matrix/) | Medium | 8-directional BFS. |
| [Cheapest Flights Within K Stops](https://leetcode.com/problems/cheapest-flights-within-k-stops/) | Medium | BFS (Level limits) or Bellman-Ford. |