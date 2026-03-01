## The Bellman-Ford Algorithm

The Bellman-Ford algorithm is another method for solving the **Single-Source Shortest Path (SSSP)** problem. Its key advantage over Dijkstra's algorithm is its ability to handle graphs with **negative edge weights**.

Because of its ability to handle negative weights, Bellman-Ford can also be used to detect **negative-weight cycles**—a cycle in a graph whose edges sum to a negative value. If such a cycle is reachable from the source, there is no "shortest" path, as you can loop through the cycle infinitely to make the path cost arbitrarily small.

### The Core Idea
Bellman-Ford is a dynamic programming-based algorithm. It is simpler than Dijkstra's but less efficient. The core idea is to progressively "relax" all the edges in the graph.

An "edge relaxation" is the process of checking if the path to a node `v` can be shortened by going through another node `u`.
`if distance[u] + weight(u, v) < distance[v]: distance[v] = distance[u] + weight(u, v)`

The algorithm repeats this relaxation step for every edge in the graph `V-1` times (where `V` is the number of vertices). After `k` iterations, the algorithm is guaranteed to have found the shortest path that uses at most `k` edges. Since a simple shortest path can have at most `V-1` edges, iterating `V-1` times guarantees finding the shortest path.

### The Algorithm
1.  **Initialization**:
    - Create a `distances` array. Initialize the distance to the `source` node as 0 and all other nodes to infinity.

2.  **Relax Edges**:
    - Loop `V-1` times:
      - For each `edge (u, v)` with weight `w` in the graph:
        - If `distances[u] + w < distances[v]`, then update `distances[v] = distances[u] + w`.

3.  **Check for Negative-Weight Cycles**:
    - After `V-1` iterations, perform one final iteration over all edges.
    - If for any `edge (u, v)` with weight `w`, we can still relax the edge (`distances[u] + w < distances[v]`), it means a shorter path is still possible. This can only happen if a negative-weight cycle exists. You can report that a negative cycle is present and terminate.

### Complexity
- **Time Complexity**: **O(V * E)**, where V is the number of vertices and E is the number of edges. The outer loop runs V-1 times, and the inner loop iterates through all E edges. This is significantly slower than Dijkstra's O(E log V) on graphs without negative edges.
- **Space Complexity**: **O(V)** to store the distances array.

### Implementation

>[!example]- C++
>```cpp
>#include <vector>
>#include <unordered_map>
>#include <limits>
>#include <iostream>
>using namespace std;
>
>unordered_map<int, int> bellmanFord(vector<tuple<int, int, int>>& edges, int numVertices, int source) {
>    // edges: list of tuples (u, v, weight)
>
>    unordered_map<int, int> distances;
>
>    // Step 1: Initialize distances
>    for (int i = 0; i < numVertices; i++) {
>        distances[i] = numeric_limits<int>::max();
>    }
>    distances[source] = 0;
>
>    // Step 2: Relax edges V-1 times
>    for (int i = 0; i < numVertices - 1; i++) {
>        for (const auto& [u, v, weight] : edges) {
>            if (distances[u] != numeric_limits<int>::max() &&
>                distances[u] + weight < distances[v]) {
>                distances[v] = distances[u] + weight;
>            }
>        }
>    }
>
>    // Step 3: Check for negative-weight cycles
>    for (const auto& [u, v, weight] : edges) {
>        if (distances[u] != numeric_limits<int>::max() &&
>            distances[u] + weight < distances[v]) {
>            cout << "Graph contains a negative-weight cycle" << endl;
>            return {}; // Return empty map
>        }
>    }
>
>    return distances;
>}
>```

>[!example]- Java
>```java
>import java.util.*;
>
>public class BellmanFord {
>    public static Map<Integer, Integer> bellmanFord(List<int[]> edges, int numVertices, int source) {
>        // edges: list of arrays [u, v, weight]
>
>        Map<Integer, Integer> distances = new HashMap<>();
>
>        // Step 1: Initialize distances
>        for (int i = 0; i < numVertices; i++) {
>            distances.put(i, Integer.MAX_VALUE);
>        }
>        distances.put(source, 0);
>
>        // Step 2: Relax edges V-1 times
>        for (int i = 0; i < numVertices - 1; i++) {
>            for (int[] edge : edges) {
>                int u = edge[0];
>                int v = edge[1];
>                int weight = edge[2];
>
>                if (distances.get(u) != Integer.MAX_VALUE &&
>                    distances.get(u) + weight < distances.get(v)) {
>                    distances.put(v, distances.get(u) + weight);
>                }
>            }
>        }
>
>        // Step 3: Check for negative-weight cycles
>        for (int[] edge : edges) {
>            int u = edge[0];
>            int v = edge[1];
>            int weight = edge[2];
>
>            if (distances.get(u) != Integer.MAX_VALUE &&
>                distances.get(u) + weight < distances.get(v)) {
>                System.out.println("Graph contains a negative-weight cycle");
>                return null;
>            }
>        }
>
>        return distances;
>    }
>}
>```

>[!example]- Python
>```python
>def bellman_ford(graph, num_vertices, source):
>    # graph should be a list of edges: [(u, v, weight), ...]
>
>    # Step 1: Initialize distances
>    distances = {i: float('inf') for i in range(num_vertices)}
>    distances[source] = 0
>
>    # Step 2: Relax edges V-1 times
>    for _ in range(num_vertices - 1):
>        for u, v, weight in graph:
>            if distances[u] != float('inf') and distances[u] + weight < distances[v]:
>                distances[v] = distances[u] + weight
>
>    # Step 3: Check for negative-weight cycles
>    for u, v, weight in graph:
>        if distances[u] != float('inf') and distances[u] + weight < distances[v]:
>            print("Graph contains a negative-weight cycle")
>            return None # Or handle as needed
>
>    return distances
>```

>[!example]- JavaScript
>```javascript
>function bellmanFord(edges, numVertices, source) {
>    // edges: array of arrays [[u, v, weight], ...]
>
>    const distances = {};
>
>    // Step 1: Initialize distances
>    for (let i = 0; i < numVertices; i++) {
>        distances[i] = Infinity;
>    }
>    distances[source] = 0;
>
>    // Step 2: Relax edges V-1 times
>    for (let i = 0; i < numVertices - 1; i++) {
>        for (const [u, v, weight] of edges) {
>            if (distances[u] !== Infinity && distances[u] + weight < distances[v]) {
>                distances[v] = distances[u] + weight;
>            }
>        }
>    }
>
>    // Step 3: Check for negative-weight cycles
>    for (const [u, v, weight] of edges) {
>        if (distances[u] !== Infinity && distances[u] + weight < distances[v]) {
>            console.log("Graph contains a negative-weight cycle");
>            return null;
>        }
>    }
>
>    return distances;
>}
>```

While slower than Dijkstra's, Bellman-Ford's robustness makes it essential for problems where negative edge weights are a possibility.
