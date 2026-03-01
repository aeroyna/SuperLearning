# Graph Algorithms

Graph algorithms solve problems involving networks, relationships, and connectivity. This section covers the essential graph algorithms for interviews: shortest paths, minimum spanning trees, and topological ordering.

## Overview

Core graph algorithm categories:
- **Shortest Path**: Find minimum-cost paths between vertices
- **Minimum Spanning Tree**: Connect all vertices with minimum total edge weight
- **Topological Sort**: Order vertices respecting dependencies

## Topics

- [18. Shortest Path Algorithms](Shortest_Path/00_shortest_path.md)
  - Dijkstra's Algorithm
  - Bellman-Ford Algorithm
  - Floyd-Warshall Algorithm

- [19. Minimum Spanning Trees](Minimum_Spanning_Tree/00_minimum_spanning_tree.md)
  - Kruskal's Algorithm
  - Prim's Algorithm
  - Union-Find

- [20. Topological Sort](Topological_Sort/00_topological_sort.md)
  - Kahn's Algorithm (BFS)
  - DFS-based approach

## Algorithm Selection Guide

```
Problem Type                    Algorithm              Requirements
───────────────────────────────────────────────────────────────────────
Shortest path (unweighted)      BFS                   O(V + E)
Shortest path (non-negative)    Dijkstra              O((V+E) log V)
Shortest path (negative edges)  Bellman-Ford          O(VE)
All pairs shortest path         Floyd-Warshall        O(V³)
Minimum spanning tree (sparse)  Kruskal               O(E log E)
Minimum spanning tree (dense)   Prim                  O(E log V)
Dependency ordering             Topological Sort      O(V + E)
Cycle detection (directed)      DFS colors / Topo     O(V + E)
Connectivity                    Union-Find            O(α(n)) per op
```

## Complexity Summary

| Algorithm | Time | Space | When to Use |
|-----------|------|-------|-------------|
| BFS | O(V + E) | O(V) | Unweighted shortest path |
| Dijkstra | O((V+E) log V) | O(V) | Non-negative weights |
| Bellman-Ford | O(VE) | O(V) | Negative weights, detect negative cycles |
| Floyd-Warshall | O(V³) | O(V²) | All pairs, dense graph |
| Kruskal | O(E log E) | O(V) | Sparse graphs, edge list |
| Prim | O(E log V) | O(V) | Dense graphs, adjacency list |
| Topological Sort | O(V + E) | O(V) | DAG ordering |

## Quick Reference

### When to Use Dijkstra vs Bellman-Ford

| Scenario | Algorithm |
|----------|-----------|
| All edges non-negative | Dijkstra (faster) |
| Some edges negative | Bellman-Ford |
| Need to detect negative cycle | Bellman-Ford |
| Dense graph | Consider Floyd-Warshall |

### When to Use Kruskal vs Prim

| Scenario | Algorithm |
|----------|-----------|
| Sparse graph | Kruskal |
| Dense graph | Prim |
| Edges given as list | Kruskal |
| Adjacency list | Prim |
| Need incremental additions | Union-Find / Prim |

## Common Graph Problem Types

### Interview Frequency

1. **High Frequency**
   - BFS/DFS traversal
   - Shortest path in grid/graph
   - Cycle detection
   - Connected components

2. **Medium Frequency**
   - Topological sort
   - Dijkstra's shortest path
   - Union-Find problems

3. **Lower Frequency (but important)**
   - Bellman-Ford
   - Minimum spanning tree
   - Floyd-Warshall

## Common Pitfalls

1. **Wrong algorithm for edge weights**: Dijkstra fails with negative edges
2. **Forgetting visited set**: Infinite loops in cyclic graphs
3. **Self-loops and multi-edges**: Clarify if allowed
4. **Directed vs undirected**: Different algorithms/handling
5. **Disconnected graphs**: May need to run from all vertices

## Key Interview Problems

| Problem | Algorithm | Difficulty | LeetCode Link |
| --------- | ----------- | ------------ | --- |
| Network Delay Time | Dijkstra | Medium | [Link](https://leetcode.com/problems/network-delay-time/) |
| Cheapest Flights Within K Stops | Modified Dijkstra/BF | Medium | [Link](https://leetcode.com/problems/cheapest-flights-within-k-stops/) |
| Course Schedule II | Topological Sort | Medium | [Link](https://leetcode.com/problems/course-schedule-ii/) |
| Find the City With Smallest Number of Neighbors | Floyd-Warshall | Medium | [Link](https://leetcode.com/problems/find-the-city-with-smallest-number-of-neighbors/) |
| Min Cost to Connect All Points | MST (Kruskal/Prim) | Medium | [Link](https://leetcode.com/problems/min-cost-to-connect-all-points/) |
| Redundant Connection | Union-Find | Medium | [Link](https://leetcode.com/problems/redundant-connection/) |
