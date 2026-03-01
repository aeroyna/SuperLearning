## The Floyd-Warshall Algorithm

The Floyd-Warshall algorithm solves the **All-Pairs Shortest Path (APSP)** problem. It is designed to find the shortest paths between *every pair* of vertices in a weighted directed graph. It is a dynamic programming-based algorithm.

Like Bellman-Ford, Floyd-Warshall can handle **negative edge weights**, but it cannot handle negative-weight cycles (it can be used to detect them).

### The Core Idea
The algorithm is remarkably simple. It works by incrementally allowing vertices to be used as intermediate points in paths.

It considers all possible paths between any two vertices `i` and `j`. A path can either be the direct edge `(i, j)` or it can go through some set of intermediate vertices. Floyd-Warshall systematically considers each vertex `k` and checks if the path from `i` to `j` can be shortened by going through `k`.

The core recurrence relation is:
`dist(i, j) = min(dist(i, j), dist(i, k) + dist(k, j))`

This means the shortest path from `i` to `j` is either the current known shortest path, or a new path that goes from `i` to `k` and then from `k` to `j`. The algorithm tries this for every possible `k`.

### The Algorithm
1.  **Initialization**:
    - Create a 2D `dist` matrix of size `V x V`, where `V` is the number of vertices.
    - Initialize `dist[i][j]` with the weight of the direct edge from `i` to `j`.
    - If there is no direct edge from `i` to `j`, set `dist[i][j]` to infinity.
    - For all vertices `i`, set `dist[i][i] = 0`.

2.  **Main Loops**:
    - Iterate through all possible intermediate vertices `k` from `0` to `V-1`.
      - For all source vertices `i` from `0` to `V-1`.
        - For all destination vertices `j` from `0` to `V-1`.
          - Perform the relaxation step:
            `dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j])`

3.  **Termination**: After the loops complete, the `dist` matrix will contain the shortest path distances between all pairs of vertices.

4.  **(Optional) Negative Cycle Detection**: After the algorithm runs, if any `dist[i][i]` is negative, it means that vertex `i` is part of a negative-weight cycle.

### Complexity
- **Time Complexity**: **O(V^3)**. The three nested loops make this algorithm cubic in the number of vertices. This makes it suitable only for smaller graphs.
- **Space Complexity**: **O(V^2)** to store the distance matrix.

### Implementation

>[!example]- C++
>```cpp
>#include <vector>
>#include <limits>
>#include <algorithm>
>#include <iostream>
>using namespace std;
>
>vector<vector<int>> floydWarshall(vector<tuple<int, int, int>>& edges, int numVertices) {
>    // edges: list of tuples (u, v, weight)
>
>    const int INF = numeric_limits<int>::max() / 2; // Avoid overflow
>
>    // Step 1: Initialize distance matrix
>    vector<vector<int>> dist(numVertices, vector<int>(numVertices, INF));
>
>    for (int i = 0; i < numVertices; i++) {
>        dist[i][i] = 0;
>    }
>
>    for (const auto& [u, v, weight] : edges) {
>        dist[u][v] = weight;
>    }
>
>    // Step 2: Main loops
>    for (int k = 0; k < numVertices; k++) {
>        for (int i = 0; i < numVertices; i++) {
>            for (int j = 0; j < numVertices; j++) {
>                // Relaxation step
>                if (dist[i][k] != INF && dist[k][j] != INF) {
>                    dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j]);
>                }
>            }
>        }
>    }
>
>    // Step 3: Check for negative cycles
>    for (int i = 0; i < numVertices; i++) {
>        if (dist[i][i] < 0) {
>            cout << "Graph contains a negative-weight cycle" << endl;
>            return {};
>        }
>    }
>
>    return dist;
>}
>```

>[!example]- Java
>```java
>import java.util.*;
>
>public class FloydWarshall {
>    public static int[][] floydWarshall(List<int[]> edges, int numVertices) {
>        // edges: list of arrays [u, v, weight]
>
>        final int INF = Integer.MAX_VALUE / 2; // Avoid overflow
>
>        // Step 1: Initialize distance matrix
>        int[][] dist = new int[numVertices][numVertices];
>
>        for (int i = 0; i < numVertices; i++) {
>            Arrays.fill(dist[i], INF);
>            dist[i][i] = 0;
>        }
>
>        for (int[] edge : edges) {
>            int u = edge[0];
>            int v = edge[1];
>            int weight = edge[2];
>            dist[u][v] = weight;
>        }
>
>        // Step 2: Main loops
>        for (int k = 0; k < numVertices; k++) {
>            for (int i = 0; i < numVertices; i++) {
>                for (int j = 0; j < numVertices; j++) {
>                    // Relaxation step
>                    if (dist[i][k] != INF && dist[k][j] != INF) {
>                        dist[i][j] = Math.min(dist[i][j], dist[i][k] + dist[k][j]);
>                    }
>                }
>            }
>        }
>
>        // Step 3: Check for negative cycles
>        for (int i = 0; i < numVertices; i++) {
>            if (dist[i][i] < 0) {
>                System.out.println("Graph contains a negative-weight cycle");
>                return null;
>            }
>        }
>
>        return dist;
>    }
>}
>```

>[!example]- Python
>```python
>def floyd_warshall(graph, num_vertices):
>    # graph should be a list of edges: [(u, v, weight), ...]
>    # Or an adjacency matrix could be passed in directly.
>
>    # Step 1: Initialize distance matrix
>    dist = [[float('inf')] * num_vertices for _ in range(num_vertices)]
>
>    for i in range(num_vertices):
>        dist[i][i] = 0
>
>    for u, v, weight in graph:
>        dist[u][v] = weight
>
>    # Step 2: Main loops
>    for k in range(num_vertices):
>        for i in range(num_vertices):
>            for j in range(num_vertices):
>                # Relaxation step
>                if dist[i][k] != float('inf') and dist[k][j] != float('inf'):
>                    dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j])
>
>    # Step 3 (Optional): Check for negative cycles
>    for i in range(num_vertices):
>        if dist[i][i] < 0:
>            print("Graph contains a negative-weight cycle")
>            return None
>
>    return dist
>```

>[!example]- JavaScript
>```javascript
>function floydWarshall(edges, numVertices) {
>    // edges: array of arrays [[u, v, weight], ...]
>
>    // Step 1: Initialize distance matrix
>    const dist = Array(numVertices).fill(null).map(() =>
>        Array(numVertices).fill(Infinity)
>    );
>
>    for (let i = 0; i < numVertices; i++) {
>        dist[i][i] = 0;
>    }
>
>    for (const [u, v, weight] of edges) {
>        dist[u][v] = weight;
>    }
>
>    // Step 2: Main loops
>    for (let k = 0; k < numVertices; k++) {
>        for (let i = 0; i < numVertices; i++) {
>            for (let j = 0; j < numVertices; j++) {
>                // Relaxation step
>                if (dist[i][k] !== Infinity && dist[k][j] !== Infinity) {
>                    dist[i][j] = Math.min(dist[i][j], dist[i][k] + dist[k][j]);
>                }
>            }
>        }
>    }
>
>    // Step 3: Check for negative cycles
>    for (let i = 0; i < numVertices; i++) {
>        if (dist[i][i] < 0) {
>            console.log("Graph contains a negative-weight cycle");
>            return null;
>        }
>    }
>
>    return dist;
>}
>```

Because of its high time complexity, Floyd-Warshall is less common in interviews than SSSP algorithms, but it's a powerful tool for the specific problem it solves.
