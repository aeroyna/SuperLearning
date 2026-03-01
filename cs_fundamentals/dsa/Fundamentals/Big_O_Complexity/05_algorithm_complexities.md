## Time Complexity of Common Algorithms

This cheatsheet provides the time complexities for common algorithms, which are essential building blocks for solving more complex problems.

---

### Sorting Algorithms

| Algorithm         | Best Case    | Average Case | Worst Case   | Notes                                           |
| ----------------- | :----------: | :----------: | :----------: | ----------------------------------------------- |
| **Bubble Sort**   |     O(n)     |    O(n^2)    |    O(n^2)    | Inefficient, rarely used in practice.           |
| **Insertion Sort**|     O(n)     |    O(n^2)    |    O(n^2)    | Efficient for small or nearly sorted datasets.  |
| **Selection Sort**|    O(n^2)    |    O(n^2)    |    O(n^2)    |                                                 |
| **Merge Sort**    |  O(n log n)  |  O(n log n)  |  O(n log n)  | Stable, but requires O(n) extra space.          |
| **Quick Sort**    |  O(n log n)  |  O(n log n)  |    O(n^2)    | Not stable, but in-place with O(log n) space.   |
| **Heap Sort**     |  O(n log n)  |  O(n log n)  |  O(n log n)  | In-place with O(1) space, but not stable.       |
| **Counting Sort** |   O(n + k)   |   O(n + k)   |   O(n + k)   | Non-comparison. `k` is the range of input values. |
| **Radix Sort**    |   O(d(n+b))  |   O(d(n+b))  |   O(d(n+b))  | Non-comparison. `d` is digits, `b` is base.     |

---

### Graph Algorithms

V = Number of Vertices, E = Number of Edges

| Algorithm           | Time Complexity        | Notes                                                        |
| ------------------- | :--------------------: | ------------------------------------------------------------ |
| **BFS (Traversal)** |        O(V + E)        | Uses a queue. Finds shortest path on unweighted graphs.      |
| **DFS (Traversal)** |        O(V + E)        | Uses a stack (or recursion). Used for pathfinding, cycles.   |
| **Dijkstra's**      |      O(E log V)        | With a min-priority queue (heap). For SSSP on graphs with non-negative weights. |
| **Bellman-Ford**    |        O(V * E)        | SSSP. Slower, but handles negative edge weights.             |
| **Floyd-Warshall**  |         O(V^3)         | All-Pairs Shortest Path (APSP). Handles negative weights.    |
| **Kruskal's (MST)** |      O(E log E)        | Greedy. Uses Union-Find. Sorts edges first.                  |
| **Prim's (MST)**    |      O(E log V)        | Greedy. Uses a min-priority queue (heap). Grows a single tree. |
| **Topological Sort**|        O(V + E)        | For Directed Acyclic Graphs (DAGs). Can use BFS (Kahn's) or DFS. |

---

### Other Common Algorithms

| Algorithm                 | Time Complexity | Notes                                                        |
| ------------------------- | :-------------: | ------------------------------------------------------------ |
| **Binary Search**         |    O(log n)     | Requires the data to be sorted.                              |
| **Backtracking**          |   Exponential   | Often O(k^n) or O(n!). The exact complexity depends on the number of choices and the depth of recursion. |
| **Dynamic Programming**   |    Varies       | Typically `(Number of States) * (Time per State)`. Example: O(n*m) for LCS. |
