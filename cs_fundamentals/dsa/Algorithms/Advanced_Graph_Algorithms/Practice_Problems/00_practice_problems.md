# Practice Problems: Advanced Graph Algorithms

Shortest paths, Minimum Spanning Trees, and Network Flow.

## Shortest Paths

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Network Delay Time](https://leetcode.com/problems/network-delay-time/) | Medium | Dijkstra's Algorithm (non-negative weights). |
| [Cheapest Flights Within K Stops](https://leetcode.com/problems/cheapest-flights-within-k-stops/) | Medium | Bellman-Ford or Dijkstra with state (city, stops). |
| [Path With Minimum Effort](https://leetcode.com/problems/path-with-minimum-effort/) | Medium | Modified Dijkstra (min-max weight). |
| [Find the City With the Smallest Number of Neighbors at a Threshold Distance](https://leetcode.com/problems/find-the-city-with-the-smallest-number-of-neighbors-at-a-threshold-distance/) | Medium | Floyd-Warshall (all-pairs shortest path). |

## Minimum Spanning Tree (MST)

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Min Cost to Connect All Points](https://leetcode.com/problems/min-cost-to-connect-all-points/) | Medium | Prim's or Kruskal's Algorithm. |
| [Connecting Cities With Minimum Cost](https://leetcode.com/problems/connecting-cities-with-minimum-cost/) | Medium | Standard MST. |
| [Optimize Water Distribution in a Village](https://leetcode.com/problems/optimize-water-distribution-in-a-village/) | Hard | Add virtual node 0 connected to all houses with well cost. Run MST. |

## Strongly Connected Components & Bridges

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Critical Connections in a Network](https://leetcode.com/problems/critical-connections-in-a-network/) | Hard | Tarjan's Bridge finding algorithm (DFS low-link values). |
