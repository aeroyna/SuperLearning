# Shortest Path Algorithms

Shortest path algorithms find the minimum-cost route between vertices in a weighted graph. Different algorithms handle different constraints: non-negative weights, negative weights, single-source vs all-pairs.

## Overview

Algorithm selection based on graph properties:
- **Unweighted**: BFS - O(V + E)
- **Non-negative weights**: Dijkstra - O((V+E) log V)
- **Negative weights**: Bellman-Ford - O(VE)
- **All pairs**: Floyd-Warshall - O(V³)

## Topics

- [18.1 Dijkstra's Algorithm](01_dijkstras_algorithm.md)
- [18.2 Bellman-Ford Algorithm](02_bellman_ford.md)
- [18.3 Floyd-Warshall Algorithm](03_floyd_warshall.md)

## Dijkstra's Algorithm

Greedy algorithm for non-negative edge weights. Uses a priority queue to always process the vertex with minimum distance.

>[!example]- C++
>```cpp
>unordered_map<int, int> dijkstra(unordered_map<int, vector<pair<int, int>>>& graph, int start) {
>    unordered_map<int, int> distances;
>    for (auto const& [node, _] : graph) distances[node] = INT_MAX;
>    distances[start] = 0;
>    
>    // min-heap: {distance, node}
>    priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> pq;
>    pq.push({0, start});
>    
>    while (!pq.empty()) {
>        auto [dist, node] = pq.top();
>        pq.pop();
>        
>        if (dist > distances[node]) continue;
>        
>        for (auto const& [neighbor, weight] : graph[node]) {
>            int newDist = dist + weight;
>            if (newDist < distances[neighbor]) {
>                distances[neighbor] = newDist;
>                pq.push({newDist, neighbor});
>            }
>        }
>    }
>    return distances;
>}
>```

>[!example]- Java
>```java
>public Map<Integer, Integer> dijkstra(Map<Integer, List<int[]>> graph, int start) {
>    Map<Integer, Integer> distances = new HashMap<>();
>    for (Integer node : graph.keySet()) distances.put(node, Integer.MAX_VALUE);
>    distances.put(start, 0);
>    
>    // min-heap: {distance, node}
>    PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]);
>    pq.offer(new int[]{0, start});
>    
>    while (!pq.isEmpty()) {
>        int[] current = pq.poll();
>        int dist = current[0];
>        int node = current[1];
>        
>        if (dist > distances.get(node)) continue;
>        
>        for (int[] edge : graph.get(node)) {
>            int neighbor = edge[0];
>            int weight = edge[1];
>            int newDist = dist + weight;
>            if (newDist < distances.get(neighbor)) {
>                distances.put(neighbor, newDist);
>                pq.offer(new int[]{newDist, neighbor});
>            }
>        }
>    }
>    return distances;
>}
>```

>[!example]- Python
>```python
>import heapq
>
>def dijkstra(graph, start):
>    """
>    graph: adjacency list {node: [(neighbor, weight), ...]}
>    Returns: dict of shortest distances from start
>    """
>    distances = {node: float('inf') for node in graph}
>    distances[start] = 0
>    pq = [(0, start)]  # (distance, node)
>
>    while pq:
>        dist, node = heapq.heappop(pq)
>
>        # Skip if we've found a better path
>        if dist > distances[node]:
>            continue
>
>        for neighbor, weight in graph[node]:
>            new_dist = dist + weight
>            if new_dist < distances[neighbor]:
>                distances[neighbor] = new_dist
>                heapq.heappush(pq, (new_dist, neighbor))
>
>    return distances
>```

>[!example]- JavaScript
>```javascript
>// Requires a PriorityQueue implementation (e.g. MinPriorityQueue)
>function dijkstra(graph, start) {
>    const distances = {};
>    for (const node in graph) distances[node] = Infinity;
>    distances[start] = 0;
>    
>    // Assuming MinPriorityQueue exists, stores {element: node, priority: distance}
>    const pq = new MinPriorityQueue();
>    pq.enqueue(start, 0);
>    
>    while (!pq.isEmpty()) {
>        const { element: node, priority: dist } = pq.dequeue();
>        
>        if (dist > distances[node]) continue;
>        
>        for (const [neighbor, weight] of graph[node]) {
>            const newDist = dist + weight;
>            if (newDist < distances[neighbor]) {
>                distances[neighbor] = newDist;
>                pq.enqueue(neighbor, newDist);
>            }
>        }
>    }
>    return distances;
>}
>```

**Why it works**: With non-negative weights, once we pop a vertex, we've found its shortest path (greedy choice property).

**Why it fails with negative edges**: A "longer" path through unvisited vertices might become shorter after discovering negative edges.

### Dijkstra Complexity

- Time: O((V + E) log V) with binary heap
- Space: O(V) for distances and priority queue

## Bellman-Ford Algorithm

Handles negative edges by relaxing all edges V-1 times. Can detect negative cycles.

>[!example]- C++
>```cpp
>// Returns empty map if negative cycle detected
>unordered_map<int, int> bellmanFord(vector<tuple<int, int, int>>& edges, int n, int start) {
>    vector<int> distances(n, INT_MAX);
>    distances[start] = 0;
>    
>    // Relax edges V-1 times
>    for (int i = 0; i < n - 1; i++) {
>        for (const auto& [u, v, weight] : edges) {
>            if (distances[u] != INT_MAX && distances[u] + weight < distances[v]) {
>                distances[v] = distances[u] + weight;
>            }
>        }
>    }
>    
>    // Check for negative cycles
>    for (const auto& [u, v, weight] : edges) {
>        if (distances[u] != INT_MAX && distances[u] + weight < distances[v]) {
>            return {}; // Negative cycle detected
>        }
>    }
>    
>    unordered_map<int, int> result;
>    for(int i=0; i<n; i++) result[i] = distances[i];
>    return result;
>}
>```

>[!example]- Java
>```java
>public int[] bellmanFord(int[][] edges, int n, int start) {
>    int[] distances = new int[n];
>    Arrays.fill(distances, Integer.MAX_VALUE);
>    distances[start] = 0;
>    
>    // Relax edges V-1 times
>    for (int i = 0; i < n - 1; i++) {
>        for (int[] edge : edges) {
>            int u = edge[0], v = edge[1], weight = edge[2];
>            if (distances[u] != Integer.MAX_VALUE && distances[u] + weight < distances[v]) {
>                distances[v] = distances[u] + weight;
>            }
>        }
>    }
>    
>    // Check for negative cycles
>    for (int[] edge : edges) {
>        int u = edge[0], v = edge[1], weight = edge[2];
>        if (distances[u] != Integer.MAX_VALUE && distances[u] + weight < distances[v]) {
>            return null; // Negative cycle detected
>        }
>    }
>    
>    return distances;
>}
>```

>[!example]- Python
>```python
>def bellman_ford(graph, n, start):
>    """
>    graph: list of edges [(u, v, weight), ...]
>    n: number of vertices (0 to n-1)
>    Returns: distances dict, or None if negative cycle exists
>    """
>    distances = [float('inf')] * n
>    distances[start] = 0
>
>    # Relax all edges V-1 times
>    for _ in range(n - 1):
>        for u, v, weight in graph:
>            if distances[u] != float('inf') and distances[u] + weight < distances[v]:
>                distances[v] = distances[u] + weight
>
>    # Check for negative cycles
>    for u, v, weight in graph:
>        if distances[u] != float('inf') and distances[u] + weight < distances[v]:
>            return None  # Negative cycle detected
>
>    return distances
>```

>[!example]- JavaScript
>```javascript
>function bellmanFord(edges, n, start) {
>    const distances = new Array(n).fill(Infinity);
>    distances[start] = 0;
>    
>    // Relax edges V-1 times
>    for (let i = 0; i < n - 1; i++) {
>        for (const [u, v, weight] of edges) {
>            if (distances[u] !== Infinity && distances[u] + weight < distances[v]) {
>                distances[v] = distances[u] + weight;
>            }
>        }
>    }
>    
>    // Check for negative cycles
>    for (const [u, v, weight] of edges) {
>        if (distances[u] !== Infinity && distances[u] + weight < distances[v]) {
>            return null; // Negative cycle detected
>        }
>    }
>    
>    return distances;
>}
>```

**Why V-1 iterations**: Shortest path has at most V-1 edges. Each iteration propagates shortest path by one edge.

### Bellman-Ford Complexity

- Time: O(VE)
- Space: O(V)

## Floyd-Warshall Algorithm

All-pairs shortest paths using dynamic programming.

>[!example]- C++
>```cpp
>vector<vector<long long>> floydWarshall(int n, vector<tuple<int, int, int>>& edges) {
>    const long long INF = 1e18; // Use large enough value
>    vector<vector<long long>> dist(n, vector<long long>(n, INF));
>
>    // Initialize
>    for (int i = 0; i < n; i++) dist[i][i] = 0;
>    for (const auto& [u, v, w] : edges) {
>        dist[u][v] = min(dist[u][v], (long long)w);
>    }
>
>    // DP: try each vertex k as intermediate
>    for (int k = 0; k < n; k++) {
>        for (int i = 0; i < n; i++) {
>            for (int j = 0; j < n; j++) {
>                if (dist[i][k] != INF && dist[k][j] != INF) {
>                    dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j]);
>                }
>            }
>        }
>    }
>    return dist;
>}
>```

>[!example]- Java
>```java
>public long[][] floydWarshall(int n, int[][] edges) {
>    long INF = Long.MAX_VALUE / 2; // Prevent overflow
>    long[][] dist = new long[n][n];
>    for (long[] row : dist) Arrays.fill(row, INF);
>
>    // Initialize
>    for (int i = 0; i < n; i++) dist[i][i] = 0;
>    for (int[] edge : edges) {
>        int u = edge[0], v = edge[1], w = edge[2];
>        dist[u][v] = Math.min(dist[u][v], w);
>    }
>
>    // DP: try each vertex k as intermediate
>    for (int k = 0; k < n; k++) {
>        for (int i = 0; i < n; i++) {
>            for (int j = 0; j < n; j++) {
>                if (dist[i][k] != INF && dist[k][j] != INF) {
>                    dist[i][j] = Math.min(dist[i][j], dist[i][k] + dist[k][j]);
>                }
>            }
>        }
>    }
>    return dist;
>}
>```

>[!example]- Python
>```python
>def floyd_warshall(n, edges):
>    """
>    n: number of vertices
>    edges: list of (u, v, weight)
>    Returns: dist[i][j] = shortest path from i to j
>    """
>    INF = float('inf')
>    dist = [[INF] * n for _ in range(n)]
>
>    # Initialize
>    for i in range(n):
>        dist[i][i] = 0
>    for u, v, w in edges:
>        dist[u][v] = min(dist[u][v], w)
>
>    # DP: try each vertex k as intermediate
>    for k in range(n):
>        for i in range(n):
>            for j in range(n):
>                if dist[i][k] != INF and dist[k][j] != INF:
>                    dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j])
>
>    return dist
>```

>[!example]- JavaScript
>```javascript
>function floydWarshall(n, edges) {
>    const INF = Infinity;
>    const dist = Array.from({ length: n }, () => Array(n).fill(INF));
>
>    // Initialize
>    for (let i = 0; i < n; i++) dist[i][i] = 0;
>    for (const [u, v, w] of edges) {
>        dist[u][v] = Math.min(dist[u][v], w);
>    }
>
>    // DP: try each vertex k as intermediate
>    for (let k = 0; k < n; k++) {
>        for (let i = 0; i < n; i++) {
>            for (let j = 0; j < n; j++) {
>                if (dist[i][k] !== INF && dist[k][j] !== INF) {
>                    dist[i][j] = Math.min(dist[i][j], dist[i][k] + dist[k][j]);
>                }
>            }
>        }
>    }
>    return dist;
>}
>```

**State**: `dist[i][j]` = shortest path from i to j using only vertices 0..k as intermediates
**Transition**: `dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j])`

### Floyd-Warshall Complexity

- Time: O(V³)
- Space: O(V²)

## Algorithm Comparison

| Feature | Dijkstra | Bellman-Ford | Floyd-Warshall |
|---------|----------|--------------|----------------|
| Negative edges | No | Yes | Yes |
| Negative cycle detection | No | Yes | Yes |
| Single source | Yes | Yes | No (all pairs) |
| All pairs | Need V runs | Need V runs | Yes |
| Time (single source) | O((V+E) log V) | O(VE) | - |
| Time (all pairs) | O(V(V+E) log V) | O(V²E) | O(V³) |

## Common Variations

### Shortest Path with K Stops

```python
def cheapest_flight(n, flights, src, dst, k):
    """Bellman-Ford variant with limited iterations."""
    prices = [float('inf')] * n
    prices[src] = 0

    for _ in range(k + 1):
        temp = prices[:]
        for u, v, w in flights:
            if prices[u] != float('inf'):
                temp[v] = min(temp[v], prices[u] + w)
        prices = temp

    return prices[dst] if prices[dst] != float('inf') else -1
```

### Path Reconstruction

```python
def dijkstra_with_path(graph, start, end):
    distances = {node: float('inf') for node in graph}
    distances[start] = 0
    parent = {start: None}
    pq = [(0, start)]

    while pq:
        dist, node = heapq.heappop(pq)
        if node == end:
            break
        if dist > distances[node]:
            continue

        for neighbor, weight in graph[node]:
            new_dist = dist + weight
            if new_dist < distances[neighbor]:
                distances[neighbor] = new_dist
                parent[neighbor] = node
                heapq.heappush(pq, (new_dist, neighbor))

    # Reconstruct path
    path = []
    current = end
    while current is not None:
        path.append(current)
        current = parent.get(current)
    return path[::-1], distances[end]
```

## Common Pitfalls

1. **Dijkstra with negative weights**: Will give wrong answers
2. **Not handling disconnected vertices**: Check for infinity distances
3. **Wrong heap usage**: Python's heapq is min-heap
4. **Integer overflow**: Sum of weights might overflow

## Key Interview Problems

| Problem | Algorithm | Difficulty | LeetCode Link |
| --------- | ----------- | ------------ | --- |
| Network Delay Time | Dijkstra | Medium | [Link](https://leetcode.com/problems/network-delay-time/) |
| Cheapest Flights Within K Stops | Modified BF | Medium | [Link](https://leetcode.com/problems/cheapest-flights-within-k-stops/) |
| Path With Minimum Effort | Modified Dijkstra | Medium | [Link](https://leetcode.com/problems/path-with-minimum-effort/) |
| Find the City | Floyd-Warshall | Medium | [Link](https://leetcode.com/problems/find-the-city-with-the-smallest-number-of-neighbors-at-a-threshold-distance/) |
| Shortest Path in Binary Matrix | BFS | Medium | [Link](https://leetcode.com/problems/shortest-path-in-binary-matrix/) |
