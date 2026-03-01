## Kruskal's Algorithm

Kruskal's Algorithm is a classic greedy algorithm used to find the Minimum Spanning Tree (MST) of a connected, undirected, and weighted graph. The algorithm builds the MST by iteratively selecting the edge with the smallest weight that does not form a cycle with the edges already selected.

### The Core Idea (A Greedy Approach)
Kruskal's algorithm is greedy because at each step it makes the most locally optimal choice: it adds the cheapest available edge to the growing MST. To ensure this process results in a valid spanning tree, it must check that adding an edge does not create a cycle.

This cycle detection is handled efficiently by the **Union-Find** data structure (also known as a Disjoint-Set Union or DSU).

### The Algorithm
1.  **Sort Edges**: Create a list of all edges in the graph and sort them in non-decreasing order of their weights.

2.  **Initialize**:
    - Initialize an empty list to store the edges of the resulting MST.
    - Initialize a Union-Find data structure, with each vertex in its own separate component.

3.  **Iterate and Build**:
    - Iterate through the sorted edges `(u, v)` with weight `w`.
    - For each edge:
      a. Use the `find` operation of the Union-Find structure to check if vertices `u` and `v` are already in the same component.
      b. **If they are not in the same component**:
         i. It is safe to add this edge. Add the edge to the MST list.
         ii. Use the `union` operation to merge the components of `u` and `v`.
      c. **If they are already in the same component**:
         i. Adding this edge would create a cycle. Discard the edge and continue.

4.  **Termination**: The algorithm terminates when the MST has `V-1` edges (where `V` is the number of vertices), or when all edges have been considered.

### Complexity
- **Time Complexity**: **O(E log E)** or **O(E log V)**.
  - The dominant step is sorting the `E` edges, which takes O(E log E) time.
  - The subsequent loop involves `E` iterations, with each Union-Find operation taking nearly constant time (O(α(V)), where α is the very slow-growing inverse Ackermann function).
  - Since in a connected graph `E` can be up to `V^2`, `log E` can be up to `log(V^2) = 2 log V`. Thus, the complexity is often written as O(E log V).
- **Space Complexity**: **O(V + E)** to store the graph, the sorted edges, and the Union-Find data structure.

### Implementation

>[!example]- C++
>```cpp
>#include <vector>
>#include <algorithm>
>using namespace std;
>
>class UnionFind {
>public:
>    vector<int> parent, rank;
>
>    UnionFind(int n) {
>        parent.resize(n);
>        rank.resize(n, 1);
>        for (int i = 0; i < n; i++) {
>            parent[i] = i;
>        }
>    }
>
>    int find(int x) {
>        if (parent[x] != x) {
>            parent[x] = find(parent[x]); // Path compression
>        }
>        return parent[x];
>    }
>
>    bool unite(int x, int y) {
>        int rootX = find(x);
>        int rootY = find(y);
>
>        if (rootX == rootY) return false;
>
>        // Union by rank
>        if (rank[rootX] > rank[rootY]) {
>            parent[rootY] = rootX;
>        } else if (rank[rootX] < rank[rootY]) {
>            parent[rootX] = rootY;
>        } else {
>            parent[rootY] = rootX;
>            rank[rootX]++;
>        }
>        return true;
>    }
>};
>
>pair<vector<tuple<int, int, int>>, int> kruskalsAlgorithm(int numVertices, vector<tuple<int, int, int>>& edges) {
>    // edges: list of tuples (weight, u, v)
>
>    // 1. Sort all edges by weight
>    sort(edges.begin(), edges.end());
>
>    vector<tuple<int, int, int>> mst;
>    int totalWeight = 0;
>    UnionFind uf(numVertices);
>
>    // 3. Iterate through sorted edges
>    for (const auto& [weight, u, v] : edges) {
>        // If u and v are not already connected (no cycle)
>        if (uf.unite(u, v)) {
>            mst.push_back({u, v, weight});
>            totalWeight += weight;
>        }
>    }
>
>    // Check if a valid MST was formed
>    if (mst.size() == numVertices - 1) {
>        return {mst, totalWeight};
>    } else {
>        return {{}, INT_MAX}; // Graph is not connected
>    }
>}
>```

>[!example]- Java
>```java
>import java.util.*;
>
>class UnionFind {
>    private int[] parent;
>    private int[] rank;
>
>    public UnionFind(int n) {
>        parent = new int[n];
>        rank = new int[n];
>        for (int i = 0; i < n; i++) {
>            parent[i] = i;
>            rank[i] = 1;
>        }
>    }
>
>    public int find(int x) {
>        if (parent[x] != x) {
>            parent[x] = find(parent[x]); // Path compression
>        }
>        return parent[x];
>    }
>
>    public boolean union(int x, int y) {
>        int rootX = find(x);
>        int rootY = find(y);
>
>        if (rootX == rootY) return false;
>
>        // Union by rank
>        if (rank[rootX] > rank[rootY]) {
>            parent[rootY] = rootX;
>        } else if (rank[rootX] < rank[rootY]) {
>            parent[rootX] = rootY;
>        } else {
>            parent[rootY] = rootX;
>            rank[rootX]++;
>        }
>        return true;
>    }
>}
>
>public class KruskalsAlgorithm {
>    public static class Result {
>        List<int[]> mst;
>        int totalWeight;
>
>        Result(List<int[]> mst, int totalWeight) {
>            this.mst = mst;
>            this.totalWeight = totalWeight;
>        }
>    }
>
>    public static Result kruskalsAlgorithm(int numVertices, List<int[]> edges) {
>        // edges: list of arrays [weight, u, v]
>
>        // 1. Sort all edges by weight
>        Collections.sort(edges, Comparator.comparingInt(a -> a[0]));
>
>        List<int[]> mst = new ArrayList<>();
>        int totalWeight = 0;
>        UnionFind uf = new UnionFind(numVertices);
>
>        // 3. Iterate through sorted edges
>        for (int[] edge : edges) {
>            int weight = edge[0];
>            int u = edge[1];
>            int v = edge[2];
>
>            // If u and v are not already connected (no cycle)
>            if (uf.union(u, v)) {
>                mst.add(new int[]{u, v, weight});
>                totalWeight += weight;
>            }
>        }
>
>        // Check if a valid MST was formed
>        if (mst.size() == numVertices - 1) {
>            return new Result(mst, totalWeight);
>        } else {
>            return new Result(null, Integer.MAX_VALUE); // Graph is not connected
>        }
>    }
>}
>```

>[!example]- Python
>```python
># Assumes a UnionFind class is implemented (see 19.3 Union-Find)
># from union_find import UnionFind
def kruskals_algorithm(num_vertices, edges):
>    """
>    Finds the MST of a graph.
>    Args:
>      num_vertices: The number of vertices in the graph.
>      edges: A list of tuples, where each tuple is (weight, u, v).
>    """
>    # 1. Sort all edges by weight
>    edges.sort()
>
>    mst = []
>    total_weight = 0
>    uf = UnionFind(num_vertices) # 2. Initialize Union-Find
>
>    # 3. Iterate through sorted edges
>    for weight, u, v in edges:
>        # If u and v are not already connected (no cycle)
>        if uf.find(u) != uf.find(v):
>            # Add edge to MST and merge components
>            uf.union(u, v)
>            mst.append((u, v, weight))
>            total_weight += weight
>
>    # Check if a valid MST was formed
>    # All nodes should be in one component
>    # The number of edges should be V-1
>    if len(mst) == num_vertices - 1:
>        return mst, total_weight
>    else:
>        return None, float('inf') # Graph is not connected
>```

>[!example]- JavaScript
>```javascript
>class UnionFind {
>    constructor(n) {
>        this.parent = Array(n).fill(0).map((_, i) => i);
>        this.rank = Array(n).fill(1);
>    }
>
>    find(x) {
>        if (this.parent[x] !== x) {
>            this.parent[x] = this.find(this.parent[x]); // Path compression
>        }
>        return this.parent[x];
>    }
>
>    union(x, y) {
>        const rootX = this.find(x);
>        const rootY = this.find(y);
>
>        if (rootX === rootY) return false;
>
>        // Union by rank
>        if (this.rank[rootX] > this.rank[rootY]) {
>            this.parent[rootY] = rootX;
>        } else if (this.rank[rootX] < this.rank[rootY]) {
>            this.parent[rootX] = rootY;
>        } else {
>            this.parent[rootY] = rootX;
>            this.rank[rootX]++;
>        }
>        return true;
>    }
>}
>
>function kruskalsAlgorithm(numVertices, edges) {
>    // edges: array of arrays [[weight, u, v], ...]
>
>    // 1. Sort all edges by weight
>    edges.sort((a, b) => a[0] - b[0]);
>
>    const mst = [];
>    let totalWeight = 0;
>    const uf = new UnionFind(numVertices);
>
>    // 3. Iterate through sorted edges
>    for (const [weight, u, v] of edges) {
>        // If u and v are not already connected (no cycle)
>        if (uf.union(u, v)) {
>            mst.push([u, v, weight]);
>            totalWeight += weight;
>        }
>    }
>
>    // Check if a valid MST was formed
>    if (mst.length === numVertices - 1) {
>        return { mst, totalWeight };
>    } else {
>        return { mst: null, totalWeight: Infinity }; // Graph is not connected
>    }
>}
>```

Kruskal's is particularly effective for sparse graphs where the number of edges is much smaller than V^2.
