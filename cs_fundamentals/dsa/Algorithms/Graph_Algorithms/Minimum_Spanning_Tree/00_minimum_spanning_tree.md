# Minimum Spanning Tree

A Minimum Spanning Tree (MST) connects all vertices in a weighted, undirected graph with the minimum total edge weight. MST has exactly V-1 edges and no cycles.

## Overview

Two main approaches:
- **Kruskal's**: Sort edges, greedily add smallest that doesn't create cycle (uses Union-Find)
- **Prim's**: Grow tree from starting vertex, always add cheapest edge to tree

## Topics

- [19.1 Kruskal's Algorithm](01_kruskals_algorithm.md)
- [19.2 Prim's Algorithm](02_prims_algorithm.md)
- [19.3 Union-Find](03_union_find.md)

## Kruskal's Algorithm

Sort edges by weight, add each edge if it doesn't create a cycle.

>[!example]- C++
>```cpp
>struct DSU {
>    vector<int> parent;
>    DSU(int n) {
>        parent.resize(n);
>        iota(parent.begin(), parent.end(), 0);
>    }
>    int find(int x) {
>        if (parent[x] != x) parent[x] = find(parent[x]);
>        return parent[x];
>    }
>    bool unite(int x, int y) {
>        int rootX = find(x);
>        int rootY = find(y);
>        if (rootX != rootY) {
>            parent[rootX] = rootY;
>            return true;
>        }
>        return false;
>    }
>};
>
>pair<vector<tuple<int, int, int>>, int> kruskal(int n, vector<tuple<int, int, int>>& edges) {
>    sort(edges.begin(), edges.end(), [](const auto& a, const auto& b) {
>        return get<2>(a) < get<2>(b);
>    });
>    
>    DSU dsu(n);
>    vector<tuple<int, int, int>> mst;
>    int totalWeight = 0;
>    
>    for (const auto& [u, v, weight] : edges) {
>        if (dsu.unite(u, v)) {
>            mst.emplace_back(u, v, weight);
>            totalWeight += weight;
>            if (mst.size() == n - 1) break;
>        }
>    }
>    return {mst, totalWeight};
>}
>```

>[!example]- Java
>```java
>class UnionFind {
>    int[] parent;
>    public UnionFind(int n) {
>        parent = new int[n];
>        for (int i = 0; i < n; i++) parent[i] = i;
>    }
>    public int find(int x) {
>        if (parent[x] != x) parent[x] = find(parent[x]);
>        return parent[x];
>    }
>    public boolean union(int x, int y) {
>        int rootX = find(x);
>        int rootY = find(y);
>        if (rootX != rootY) {
>            parent[rootX] = rootY;
>            return true;
>        }
>        return false;
>    }
>}
>
>public class Kruskal {
>    public static List<int[]> kruskal(int n, int[][] edges) {
>        Arrays.sort(edges, (a, b) -> Integer.compare(a[2], b[2]));
>        UnionFind uf = new UnionFind(n);
>        List<int[]> mst = new ArrayList<>();
>        
>        for (int[] edge : edges) {
>            if (uf.union(edge[0], edge[1])) {
>                mst.add(edge);
>                if (mst.size() == n - 1) break;
>            }
>        }
>        return mst;
>    }
>}
>```

>[!example]- Python
>```python
>class UnionFind:
>    def __init__(self, n):
>        self.parent = list(range(n))
>        self.rank = [0] * n
>
>    def find(self, x):
>        if self.parent[x] != x:
>            self.parent[x] = self.find(self.parent[x])
>        return self.parent[x]
>
>    def union(self, x, y):
>        px, py = self.find(x), self.find(y)
>        if px == py:
>            return False
>        if self.rank[px] < self.rank[py]:
>            px, py = py, px
>        self.parent[py] = px
>        if self.rank[px] == self.rank[py]:
>            self.rank[px] += 1
>        return True
>
>def kruskal(n, edges):
>    """
>    n: number of vertices
>    edges: list of (u, v, weight)
>    Returns: list of MST edges, total weight
>    """
>    edges.sort(key=lambda x: x[2])  # Sort by weight
>    uf = UnionFind(n)
>    mst = []
>    total_weight = 0
>
>    for u, v, weight in edges:
>        if uf.union(u, v):  # No cycle
>            mst.append((u, v, weight))
>            total_weight += weight
>            if len(mst) == n - 1:  # MST complete
>                break
>
>    return mst, total_weight
>```

>[!example]- JavaScript
>```javascript
>class UnionFind {
>    constructor(n) {
>        this.parent = Array.from({ length: n }, (_, i) => i);
>    }
>    find(x) {
>        if (this.parent[x] !== x) this.parent[x] = this.find(this.parent[x]);
>        return this.parent[x];
>    }
>    union(x, y) {
>        const rootX = this.find(x);
>        const rootY = this.find(y);
>        if (rootX !== rootY) {
>            this.parent[rootX] = rootY;
>            return true;
>        }
>        return false;
>    }
>}
>
>function kruskal(n, edges) {
>    edges.sort((a, b) => a[2] - b[2]); // Sort by weight
>    const uf = new UnionFind(n);
>    const mst = [];
>    let totalWeight = 0;
>    
>    for (const [u, v, weight] of edges) {
>        if (uf.union(u, v)) {
>            mst.push([u, v, weight]);
>            totalWeight += weight;
>            if (mst.length === n - 1) break;
>        }
>    }
>    return { mst, totalWeight };
>}
>```

### Kruskal's Complexity

- Time: O(E log E) for sorting + O(E α(V)) for Union-Find = O(E log E)
- Space: O(V) for Union-Find

### Why Kruskal Works

**Greedy choice**: The smallest edge crossing any cut must be in some MST (Cut Property). By always choosing the smallest edge that connects two different components, we're always making a safe choice.

## Prim's Algorithm

Grow MST from a starting vertex, always adding the cheapest edge that connects the tree to a non-tree vertex.

>[!example]- C++
>```cpp
>pair<vector<tuple<int, int, int>>, int> prim(int n, vector<vector<pair<int, int>>>& graph) {
>    vector<bool> visited(n, false);
>    vector<tuple<int, int, int>> mst;
>    int totalWeight = 0;
>    
>    // Min-heap: {weight, vertex, parent}
>    priority_queue<tuple<int, int, int>, vector<tuple<int, int, int>>, greater<tuple<int, int, int>>> pq;
>    pq.push({0, 0, -1});
>    
>    while (!pq.empty() && mst.size() < n) {
>        auto [weight, u, parent] = pq.top();
>        pq.pop();
>        
>        if (visited[u]) continue;
>        visited[u] = true;
>        
>        if (parent != -1) {
>            mst.emplace_back(parent, u, weight);
>            totalWeight += weight;
>        }
>        
>        for (auto [v, w] : graph[u]) {
>            if (!visited[v]) {
>                pq.push({w, v, u});
>            }
>        }
>    }
>    return {mst, totalWeight};
>}
>```

>[!example]- Java
>```java
>public class Prims {
>    public static List<int[]> prim(int n, List<List<int[]>> graph) {
>        boolean[] visited = new boolean[n];
>        List<int[]> mst = new ArrayList<>();
>        
>        // Min-heap: {weight, vertex, parent}
>        PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> Integer.compare(a[0], b[0]));
>        pq.offer(new int[]{0, 0, -1});
>        
>        while (!pq.isEmpty() && mst.size() < n - 1) { // n-1 edges in MST
>            int[] current = pq.poll();
>            int weight = current[0];
>            int u = current[1];
>            int parent = current[2];
>            
>            if (visited[u]) continue;
>            visited[u] = true;
>            
>            if (parent != -1) {
>                mst.add(new int[]{parent, u, weight});
>            }
>            
>            for (int[] edge : graph.get(u)) {
>                int v = edge[0];
>                int w = edge[1];
>                if (!visited[v]) {
>                    pq.offer(new int[]{w, v, u});
>                }
>            }
>        }
>        return mst;
>    }
>}
>```

>[!example]- Python
>```python
>import heapq
>
>def prim(graph, n):
>    """
>    graph: adjacency list {node: [(neighbor, weight), ...]}
>    n: number of vertices
>    Returns: MST edges, total weight
>    """
>    visited = [False] * n
>    mst = []
>    total_weight = 0
>
>    # Start from vertex 0
>    pq = [(0, 0, -1)]  # (weight, vertex, parent)
>
>    while pq and len(mst) < n:
>        weight, u, parent = heapq.heappop(pq)
>
>        if visited[u]:
>            continue
>
>        visited[u] = True
>        if parent != -1:
>            mst.append((parent, u, weight))
>            total_weight += weight
>
>        for v, w in graph[u]:
>            if not visited[v]:
>                heapq.heappush(pq, (w, v, u))
>
>    return mst, total_weight
>```

>[!example]- JavaScript
>```javascript
>// Requires MinPriorityQueue
>function prim(n, graph) {
>    const visited = new Array(n).fill(false);
>    const mst = [];
>    let totalWeight = 0;
>    
>    // Priority Queue stores { element: { u, parent }, priority: weight }
>    const pq = new MinPriorityQueue();
>    pq.enqueue({ u: 0, parent: -1 }, 0);
>    
>    while (!pq.isEmpty() && mst.length < n) {
>        const { element: { u, parent }, priority: weight } = pq.dequeue();
>        
>        if (visited[u]) continue;
>        visited[u] = true;
>        
>        if (parent !== -1) {
>            mst.push([parent, u, weight]);
>            totalWeight += weight;
>        }
>        
>        for (const [v, w] of (graph[u] || [])) {
>            if (!visited[v]) {
>                pq.enqueue({ u: v, parent: u }, w);
>            }
>        }
>    }
>    return { mst, totalWeight };
>}
>```

### Prim's Complexity

- Time: O(E log V) with binary heap
- Space: O(V + E)

## Kruskal vs Prim

| Aspect | Kruskal | Prim |
|--------|---------|------|
| Best for | Sparse graphs | Dense graphs |
| Data structure | Union-Find | Priority Queue |
| Input format | Edge list | Adjacency list |
| Edge sorting | Required | Not required |
| Parallelizable | Edges can be processed in parallel | Sequential growth |

## Applications

1. **Network Design**: Minimum cost to connect all nodes
2. **Cluster Analysis**: Single-linkage clustering
3. **Approximation Algorithms**: TSP approximation
4. **Image Segmentation**: Minimum cuts

## Common Variations

### Minimum Cost to Connect All Points

```python
def min_cost_connect_points(points):
    """Prim's on complete graph defined by points."""
    n = len(points)
    visited = [False] * n
    pq = [(0, 0)]  # (cost, point_index)
    total_cost = 0
    edges_used = 0

    while pq and edges_used < n:
        cost, u = heapq.heappop(pq)
        if visited[u]:
            continue

        visited[u] = True
        total_cost += cost
        edges_used += 1

        for v in range(n):
            if not visited[v]:
                dist = abs(points[u][0] - points[v][0]) + abs(points[u][1] - points[v][1])
                heapq.heappush(pq, (dist, v))

    return total_cost
```

### Maximum Spanning Tree

Negate all weights, run MST, negate result.

```python
def max_spanning_tree(n, edges):
    negated = [(u, v, -w) for u, v, w in edges]
    mst, neg_weight = kruskal(n, negated)
    return [(u, v, -w) for u, v, w in mst], -neg_weight
```

### Second Minimum Spanning Tree

For each MST edge, find MST without that edge. Minimum of these is second MST.

## Common Pitfalls

1. **Disconnected graph**: MST doesn't exist, check if all vertices connected
2. **Directed graph**: MST is for undirected; use minimum arborescence for directed
3. **Multiple MSTs**: May exist when edges have equal weights
4. **Self-loops**: Should be ignored (add nothing useful)

## Key Interview Problems

| Problem | Approach | Difficulty | LeetCode Link |
| --------- | ---------- | ------------ | --- |
| Min Cost to Connect All Points | Prim's/Kruskal's | Medium | [Link](https://leetcode.com/problems/min-cost-to-connect-all-points/) |
| Connecting Cities With Minimum Cost | MST | Medium | [Link](https://leetcode.com/problems/connecting-cities-with-minimum-cost/) |
| Optimize Water Distribution | MST with virtual node | Hard | [Link](https://leetcode.com/problems/optimize-water-distribution-in-a-village/) |
| Find Critical and Pseudo-Critical Edges | MST analysis | Hard | [Link](https://leetcode.com/problems/find-critical-and-pseudo-critical-edges-in-minimum-spanning-tree/) |
