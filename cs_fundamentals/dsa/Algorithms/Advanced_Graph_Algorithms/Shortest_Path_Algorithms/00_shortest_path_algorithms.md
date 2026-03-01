## Introduction to Shortest Path Algorithms

Finding the shortest path between two nodes in a graph is one of the most fundamental and common problems in graph theory. The "shortest" path can mean the path with the fewest number of edges (for unweighted graphs) or the path with the minimum total weight (for weighted graphs).

While **Breadth-First Search (BFS)** is the perfect tool for finding the shortest path in an **unweighted** graph, more powerful algorithms are needed to handle graphs with weighted edges.

### The Problem
Given a weighted graph, a starting node `S`, and an ending node `E`, the goal is to find a path from `S` to `E` such that the sum of the weights of the edges along the path is minimized.

### Types of Shortest Path Problems
1.  **Single Source Shortest Path (SSSP)**: Find the shortest paths from a single source node to all other nodes in the graph.
    - **Dijkstra's Algorithm**: The classic solution for SSSP problems when all edge weights are **non-negative**.
    - **Bellman-Ford Algorithm**: A more robust (but slower) algorithm that can handle graphs with **negative edge weights**. It can also detect negative-weight cycles.

2.  **All-Pairs Shortest Path (APSP)**: Find the shortest paths between *every pair* of nodes in the graph.
    - **Floyd-Warshall Algorithm**: A dynamic programming-based algorithm that efficiently finds all-pairs shortest paths. It can also handle negative edge weights (but not negative-weight cycles).

### Choosing the Right Algorithm

| Algorithm           | Type | Edge Weights                 | Time Complexity          | Key Idea                               |
| ------------------- | ---- | ---------------------------- | ------------------------ | -------------------------------------- |
| **BFS**             | SSSP | Unweighted only              | O(V + E)                 | Level-by-level exploration.            |
| **Dijkstra's**      | SSSP | Non-negative                 | O(E log V) (with heap)   | Greedy. Always explore the "closest" unvisited node. |
| **Bellman-Ford**    | SSSP | Handles negative weights     | O(V * E)                 | DP. Relaxes all edges V-1 times.       |
| **Floyd-Warshall**  | APSP | Handles negative weights     | O(V^3)                   | DP. Considers each node as an intermediate path point. |

In an interview, if you encounter a shortest path problem on a weighted graph, Dijkstra's algorithm is often the expected solution, but you should always clarify if edge weights can be negative.
