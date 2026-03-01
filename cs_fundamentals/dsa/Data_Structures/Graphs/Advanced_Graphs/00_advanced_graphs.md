# Advanced Graph Topics

Advanced graph concepts extend beyond basic traversal to solve specific problem types: finding connected components efficiently, detecting bipartiteness, and handling complex graph structures.

## Overview

This section covers:
- Union-Find for dynamic connectivity
- Bipartite graph detection
- Strongly Connected Components
- Articulation points and bridges

## Topics

Advanced algorithms like shortest path (Dijkstra, Bellman-Ford), minimum spanning tree (Kruskal, Prim), and topological sort are covered in [Algorithms/Graph_Algorithms](../../../Algorithms/Graph_Algorithms/).

## Union-Find (Disjoint Set Union)

Efficiently tracks connected components with near-O(1) operations through path compression and union by rank.

### Implementation

>[!example]- C++
>```cpp
>class UnionFind {
>    vector<int> parent;
>    vector<int> rank;
>    int count;
>public:
>    UnionFind(int n) : count(n) {
>        parent.resize(n);
>        iota(parent.begin(), parent.end(), 0);
>        rank.resize(n, 0);
>    }
>    int find(int x) {
>        if (parent[x] != x) parent[x] = find(parent[x]);
>        return parent[x];
>    }
>    bool unite(int x, int y) {
>        int rootX = find(x);
>        int rootY = find(y);
>        if (rootX != rootY) {
>            if (rank[rootX] < rank[rootY]) swap(rootX, rootY);
>            parent[rootY] = rootX;
>            if (rank[rootX] == rank[rootY]) rank[rootX]++;
>            count--;
>            return true;
>        }
>        return false;
>    }
>    int getCount() { return count; }
>};
>```

>[!example]- Java
>```java
>class UnionFind {
>    private int[] parent;
>    private int[] rank;
>    private int count;
>    
>    public UnionFind(int n) {
>        parent = new int[n];
>        rank = new int[n];
>        count = n;
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
>            if (rank[rootX] < rank[rootY]) {
>                int temp = rootX; rootX = rootY; rootY = temp;
>            }
>            parent[rootY] = rootX;
>            if (rank[rootX] == rank[rootY]) rank[rootX]++;
>            count--;
>            return true;
>        }
>        return false;
>    }
>    public int getCount() { return count; }
>}
>```

>[!example]- Python
>```python
>class UnionFind:
>    def __init__(self, n):
>        self.parent = list(range(n))
>        self.rank = [0] * n
>        self.count = n  # Number of components
>
>    def find(self, x):
>        """Find root with path compression."""
>        if self.parent[x] != x:
>            self.parent[x] = self.find(self.parent[x])
>        return self.parent[x]
>
>    def union(self, x, y):
>        """Union by rank."""
>        px, py = self.find(x), self.find(y)
>        if px == py:
>            return False  # Already connected
>
>        if self.rank[px] < self.rank[py]:
>            px, py = py, px
>        self.parent[py] = px
>        if self.rank[px] == self.rank[py]:
>            self.rank[px] += 1
>
>        self.count -= 1
>        return True
>
>    def connected(self, x, y):
>        return self.find(x) == self.find(y)
>```

>[!example]- JavaScript
>```javascript
>class UnionFind {
>    constructor(n) {
>        this.parent = Array.from({ length: n }, (_, i) => i);
>        this.rank = new Array(n).fill(0);
>        this.count = n;
>    }
>    find(x) {
>        if (this.parent[x] !== x) this.parent[x] = this.find(this.parent[x]);
>        return this.parent[x];
>    }
>    union(x, y) {
>        let rootX = this.find(x);
>        let rootY = this.find(y);
>        if (rootX !== rootY) {
>            if (this.rank[rootX] < this.rank[rootY]) {
>                [rootX, rootY] = [rootY, rootX];
>            }
>            this.parent[rootY] = rootX;
>            if (this.rank[rootX] === this.rank[rootY]) this.rank[rootX]++;
>            this.count--;
>            return true;
>        }
>        return false;
>    }
>    getCount() { return this.count; }
>}
>```

### Why It's Fast

**Path compression**: During `find`, make every node point directly to root.

```
Before:  1 → 2 → 3 → 4 (root)
After:   1 → 4
         2 → 4
         3 → 4
```

**Union by rank**: Attach smaller tree under larger tree's root.

**Complexity**: O(α(n)) per operation where α = inverse Ackermann function ≈ 4 for practical n.

### Common Applications

```python
def count_components(n, edges):
    uf = UnionFind(n)
    for u, v in edges:
        uf.union(u, v)
    return uf.count

def find_redundant_connection(edges):
    """Find edge that creates cycle."""
    n = len(edges)
    uf = UnionFind(n)
    for u, v in edges:
        if not uf.union(u - 1, v - 1):  # Already connected
            return [u, v]
```

## Bipartite Graph Detection

A graph is bipartite if vertices can be colored with two colors such that no edge connects same-colored vertices.

### BFS Coloring

```python
def is_bipartite(graph):
    n = len(graph)
    color = [-1] * n

    for start in range(n):
        if color[start] != -1:
            continue

        queue = deque([start])
        color[start] = 0

        while queue:
            node = queue.popleft()
            for neighbor in graph[node]:
                if color[neighbor] == -1:
                    color[neighbor] = 1 - color[node]
                    queue.append(neighbor)
                elif color[neighbor] == color[node]:
                    return False

    return True
```

### DFS Coloring

```python
def is_bipartite_dfs(graph):
    n = len(graph)
    color = [-1] * n

    def dfs(node, c):
        color[node] = c
        for neighbor in graph[node]:
            if color[neighbor] == -1:
                if not dfs(neighbor, 1 - c):
                    return False
            elif color[neighbor] == c:
                return False
        return True

    for i in range(n):
        if color[i] == -1:
            if not dfs(i, 0):
                return False
    return True
```

**Applications**: Graph coloring, matching problems, detecting odd cycles.

## Strongly Connected Components (Kosaraju's Algorithm)

For directed graphs: find maximal subgraphs where every vertex is reachable from every other.

```python
def kosaraju_scc(graph):
    n = len(graph)
    visited = [False] * n
    order = []

    # Step 1: DFS to get finish order
    def dfs1(node):
        visited[node] = True
        for neighbor in graph[node]:
            if not visited[neighbor]:
                dfs1(neighbor)
        order.append(node)

    for i in range(n):
        if not visited[i]:
            dfs1(i)

    # Step 2: Build reverse graph
    reverse = [[] for _ in range(n)]
    for u in range(n):
        for v in graph[u]:
            reverse[v].append(u)

    # Step 3: DFS on reverse in reverse finish order
    visited = [False] * n
    sccs = []

    def dfs2(node, component):
        visited[node] = True
        component.append(node)
        for neighbor in reverse[node]:
            if not visited[neighbor]:
                dfs2(neighbor, component)

    for node in reversed(order):
        if not visited[node]:
            component = []
            dfs2(node, component)
            sccs.append(component)

    return sccs
```

## Articulation Points and Bridges

**Articulation point**: Vertex whose removal disconnects the graph.
**Bridge**: Edge whose removal disconnects the graph.

```python
def find_bridges(n, edges):
    graph = [[] for _ in range(n)]
    for u, v in edges:
        graph[u].append(v)
        graph[v].append(u)

    disc = [0] * n  # Discovery time
    low = [0] * n   # Lowest reachable discovery time
    visited = [False] * n
    bridges = []
    time = [0]

    def dfs(node, parent):
        visited[node] = True
        disc[node] = low[node] = time[0]
        time[0] += 1

        for neighbor in graph[node]:
            if not visited[neighbor]:
                dfs(neighbor, node)
                low[node] = min(low[node], low[neighbor])
                if low[neighbor] > disc[node]:
                    bridges.append((node, neighbor))
            elif neighbor != parent:
                low[node] = min(low[node], disc[neighbor])

    for i in range(n):
        if not visited[i]:
            dfs(i, -1)

    return bridges
```

**Key insight**: Edge (u, v) is bridge if `low[v] > disc[u]`—no back edge from v's subtree reaches u or above.

## Complexity Summary

| Algorithm | Time | Space |
|-----------|------|-------|
| Union-Find (m operations) | O(m α(n)) | O(n) |
| Bipartite check | O(V + E) | O(V) |
| Kosaraju SCC | O(V + E) | O(V + E) |
| Bridges (Tarjan) | O(V + E) | O(V) |

## Common Pitfalls

1. **Union-Find without optimizations**: Can degrade to O(n) per operation
2. **Bipartite on disconnected graph**: Must check all components
3. **Confusing SCC with connected components**: SCC is for directed graphs only
4. **Bridge detection with multi-edges**: Need to handle parallel edges specially

## Key Interview Problems

| Problem | Concept | Difficulty | LeetCode Link |
| --------- | --------- | ------------ | --- |
| Number of Connected Components | Union-Find | Medium | [Link](https://leetcode.com/problems/number-of-connected-components-in-an-undirected-graph/) |
| Redundant Connection | Cycle detection | Medium | [Link](https://leetcode.com/problems/redundant-connection/) |
| Is Graph Bipartite | Two-coloring | Medium | [Link](https://leetcode.com/problems/is-graph-bipartite/) |
| Possible Bipartition | Bipartite + constraints | Medium | [Link](https://leetcode.com/problems/possible-bipartition/) |
| Critical Connections | Bridges | Hard | [Link](https://leetcode.com/problems/critical-connections/) |
| Accounts Merge | Union-Find | Medium | [Link](https://leetcode.com/problems/accounts-merge/) |
