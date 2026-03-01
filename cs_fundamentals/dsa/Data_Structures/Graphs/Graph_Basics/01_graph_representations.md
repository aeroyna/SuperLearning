## Graph Representations

In a coding interview, a graph is almost never provided as a pre-built object. Instead, you'll receive a description of the graph, typically as a list of edges, and your first step is to convert this into a usable in-memory representation. The two most common representations are the **Adjacency List** and the **Adjacency Matrix**.

### 1. Adjacency List
An adjacency list is the most common and generally most efficient way to represent a graph, especially a **sparse graph** (where the number of edges is much smaller than the number of possible edges).

- **Structure**: It's a map or dictionary where each key is a node, and the value is a list of all nodes it's connected to (its neighbors).
- **Why?**: It provides a fast way to get all the neighbors of a given node, which is essential for traversal algorithms like DFS and BFS.

#### Building an Adjacency List from an Edge List
You are often given an input like `n` (the number of nodes) and `edges` (a list of pairs, e.g., `[[0, 1], [1, 2]]`).

>[!example]- C++
>```cpp
>#include <vector>
>#include <unordered_map>
>using namespace std;
>
>unordered_map<int, vector<int>> buildAdjacencyList(int n, vector<vector<int>>& edges, bool isDirected = false) {
>    // Using unordered_map to store adjacency list
>    unordered_map<int, vector<int>> graph;
>
>    for (const auto& edge : edges) {
>        int u = edge[0];
>        int v = edge[1];
>        graph[u].push_back(v);
>        // If the graph is undirected, the edge goes both ways
>        if (!isDirected) {
>            graph[v].push_back(u);
>        }
>    }
>
>    return graph;
>}
>
>// Example:
>// int numNodes = 4;
>// vector<vector<int>> edgeList = {{0, 1}, {1, 2}, {1, 3}};
>// auto graph = buildAdjacencyList(numNodes, edgeList);
>// The resulting adjacency list:
>// {
>//   0: [1],
>//   1: [0, 2, 3],
>//   2: [1],
>//   3: [1]
>// }
>// Now, graph[1] quickly gives you all of node 1's neighbors: [0, 2, 3]
>```

>[!example]- Java
>```java
>import java.util.*;
>
>public Map<Integer, List<Integer>> buildAdjacencyList(int n, int[][] edges, boolean isDirected) {
>    // Using HashMap to store adjacency list
>    Map<Integer, List<Integer>> graph = new HashMap<>();
>
>    for (int[] edge : edges) {
>        int u = edge[0];
>        int v = edge[1];
>
>        // Initialize lists if they don't exist
>        graph.putIfAbsent(u, new ArrayList<>());
>        graph.putIfAbsent(v, new ArrayList<>());
>
>        graph.get(u).add(v);
>        // If the graph is undirected, the edge goes both ways
>        if (!isDirected) {
>            graph.get(v).add(u);
>        }
>    }
>
>    return graph;
>}
>
>// Example:
>// int numNodes = 4;
>// int[][] edgeList = {{0, 1}, {1, 2}, {1, 3}};
>// Map<Integer, List<Integer>> graph = buildAdjacencyList(numNodes, edgeList, false);
>// The resulting adjacency list:
>// {
>//   0: [1],
>//   1: [0, 2, 3],
>//   2: [1],
>//   3: [1]
>// }
>// Now, graph.get(1) quickly gives you all of node 1's neighbors: [0, 2, 3]
>```

>[!example]- Python
>```python
>from collections import defaultdict
>
>def build_adjacency_list(n, edges, is_directed=False):
>    # A defaultdict is convenient as it handles key creation automatically
>    graph = defaultdict(list)
>
>    for u, v in edges:
>        graph[u].append(v)
>        # If the graph is undirected, the edge goes both ways
>        if not is_directed:
>            graph[v].append(u)
>
>    return graph
>
># Example:
>num_nodes = 4
>edge_list = [[0, 1], [1, 2], [1, 3]]
># Build an undirected graph
>graph = build_adjacency_list(num_nodes, edge_list)
># The resulting adjacency list:
># {
>#   0: [1],
>#   1: [0, 2, 3],
>#   2: [1],
>#   3: [1]
># }
># Now, graph[1] quickly gives you all of node 1's neighbors: [0, 2, 3]
>```

>[!example]- JavaScript
>```javascript
>function buildAdjacencyList(n, edges, isDirected = false) {
>    // Using Map to store adjacency list (could also use plain object)
>    const graph = new Map();
>
>    for (const [u, v] of edges) {
>        // Initialize arrays if they don't exist
>        if (!graph.has(u)) graph.set(u, []);
>        if (!graph.has(v)) graph.set(v, []);
>
>        graph.get(u).push(v);
>        // If the graph is undirected, the edge goes both ways
>        if (!isDirected) {
>            graph.get(v).push(u);
>        }
>    }
>
>    return graph;
>}
>
>// Example:
>// const numNodes = 4;
>// const edgeList = [[0, 1], [1, 2], [1, 3]];
>// const graph = buildAdjacencyList(numNodes, edgeList);
>// The resulting adjacency list:
>// {
>//   0: [1],
>//   1: [0, 2, 3],
>//   2: [1],
>//   3: [1]
>// }
>// Now, graph.get(1) quickly gives you all of node 1's neighbors: [0, 2, 3]
>```

- **Pros**: Space-efficient for sparse graphs. O(V+E) space. Iterating over neighbors is fast.
- **Cons**: Checking for the existence of a specific edge `(u, v)` takes O(degree(u)) time.

### 2. Adjacency Matrix
An adjacency matrix is a 2D `V x V` matrix (where V is the number of vertices). `matrix[i][j] = 1` signifies an edge from node `i` to node `j`.

- **Structure**: A 2D array.
- **Why?**: Useful for **dense graphs** where the number of edges is high. It provides an O(1) lookup to check if an edge exists between any two nodes.

- **Pros**: O(1) to check for a specific edge.
- **Cons**: Requires O(V^2) space, which is very inefficient for large, sparse graphs. Iterating over a node's neighbors takes O(V) time as you must scan the entire row.

For most interview problems, the adjacency list is the preferred representation.
