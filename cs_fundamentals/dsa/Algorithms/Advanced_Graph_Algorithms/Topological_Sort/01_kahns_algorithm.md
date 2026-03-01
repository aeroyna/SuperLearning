## Topological Sort: Kahn's Algorithm

Kahn's algorithm is an intuitive, BFS-based approach for generating a topological sort of a Directed Acyclic Graph (DAG). It works by iteratively removing nodes that have no incoming edges.

### The Core Idea
The algorithm is based on a simple principle: a task can be performed only after all its prerequisite tasks are completed. In graph terms, a node can be added to the topological order only after all nodes that point to it have already been added.

This means we can start with nodes that have no prerequisites, i.e., nodes with an **indegree of 0**. We add these nodes to our sorted list, and then we imagine "removing" them and their outgoing edges from the graph. This "removal" might decrease the indegree of their neighbors, potentially creating new nodes with an indegree of 0. We repeat this process until all nodes have been visited.

### The Algorithm
1.  **Compute Indegrees**:
    - Build an adjacency list representation of the graph.
    - At the same time, create an `indegree` array (or hash map) to store the number of incoming edges for each vertex.

2.  **Initialize the Queue**:
    - Create a queue (e.g., a `deque`).
    - Find all vertices with an indegree of 0 and enqueue them. These are the starting points of our sort.

3.  **Process Nodes**:
    - Initialize an empty list for the `topological_order` and a `count` of visited nodes to 0.
    - While the queue is not empty:
      a. Dequeue a vertex, `u`.
      b. Add `u` to the `topological_order` list.
      c. Increment the `count` of visited nodes.
      d. For each `neighbor`, `v`, of `u`:
         i. Decrement the indegree of `v`.
         ii. If the indegree of `v` becomes 0, enqueue `v`.

4.  **Check for Cycles**:
    - After the loop finishes, if the `count` of visited nodes is equal to the total number of vertices in the graph, then the `topological_order` is valid.
    - If `count` is less than the total number of vertices, it means there was a cycle in the graph, and a valid topological sort is impossible.

### Implementation

>[!example]- C++
>```cpp
>#include <vector>
>#include <queue>
>#include <unordered_map>
>using namespace std;
>
>vector<int> kahnsTopologicalSort(int numVertices, vector<pair<int, int>>& edges) {
>    // edges: list of pairs (u, v) representing directed edge from u to v
>
>    // Step 1: Compute indegrees and build adjacency list
>    unordered_map<int, vector<int>> adj;
>    unordered_map<int, int> indegree;
>
>    for (int i = 0; i < numVertices; i++) {
>        indegree[i] = 0;
>    }
>
>    for (const auto& [u, v] : edges) {
>        adj[u].push_back(v);
>        indegree[v]++;
>    }
>
>    // Step 2: Initialize the queue
>    queue<int> q;
>    for (int i = 0; i < numVertices; i++) {
>        if (indegree[i] == 0) {
>            q.push(i);
>        }
>    }
>
>    vector<int> topologicalOrder;
>
>    // Step 3: Process nodes
>    while (!q.empty()) {
>        int u = q.front();
>        q.pop();
>        topologicalOrder.push_back(u);
>
>        for (int v : adj[u]) {
>            indegree[v]--;
>            if (indegree[v] == 0) {
>                q.push(v);
>            }
>        }
>    }
>
>    // Step 4: Check for cycles
>    if (topologicalOrder.size() == numVertices) {
>        return topologicalOrder;
>    } else {
>        return {}; // Return empty vector indicating a cycle
>    }
>}
>```

>[!example]- Java
>```java
>import java.util.*;
>
>public class KahnsAlgorithm {
>    public static List<Integer> kahnsTopologicalSort(int numVertices, List<int[]> edges) {
>        // edges: list of arrays [u, v] representing directed edge from u to v
>
>        // Step 1: Compute indegrees and build adjacency list
>        Map<Integer, List<Integer>> adj = new HashMap<>();
>        Map<Integer, Integer> indegree = new HashMap<>();
>
>        for (int i = 0; i < numVertices; i++) {
>            adj.put(i, new ArrayList<>());
>            indegree.put(i, 0);
>        }
>
>        for (int[] edge : edges) {
>            int u = edge[0];
>            int v = edge[1];
>            adj.get(u).add(v);
>            indegree.put(v, indegree.get(v) + 1);
>        }
>
>        // Step 2: Initialize the queue
>        Queue<Integer> queue = new LinkedList<>();
>        for (int i = 0; i < numVertices; i++) {
>            if (indegree.get(i) == 0) {
>                queue.offer(i);
>            }
>        }
>
>        List<Integer> topologicalOrder = new ArrayList<>();
>
>        // Step 3: Process nodes
>        while (!queue.isEmpty()) {
>            int u = queue.poll();
>            topologicalOrder.add(u);
>
>            for (int v : adj.get(u)) {
>                indegree.put(v, indegree.get(v) - 1);
>                if (indegree.get(v) == 0) {
>                    queue.offer(v);
>                }
>            }
>        }
>
>        // Step 4: Check for cycles
>        if (topologicalOrder.size() == numVertices) {
>            return topologicalOrder;
>        } else {
>            return new ArrayList<>(); // Return empty list indicating a cycle
>        }
>    }
>}
>```

>[!example]- Python
>```python
>from collections import deque, defaultdict
>
>def kahns_topological_sort(num_vertices, edges):
>    """
>    Performs a topological sort using Kahn's algorithm.
>    Args:
>      num_vertices: The total number of vertices.
>      edges: A list of pairs (u, v) representing a directed edge from u to v.
>    """
>    # Step 1: Compute indegrees and build adjacency list
>    adj = defaultdict(list)
>    indegree = {i: 0 for i in range(num_vertices)}
>
>    for u, v in edges:
>        adj[u].append(v)
>        indegree[v] += 1
>
>    # Step 2: Initialize the queue
>    queue = deque([i for i in range(num_vertices) if indegree[i] == 0])
>
>    topological_order = []
>
>    # Step 3: Process nodes
>    while queue:
>        u = queue.popleft()
>        topological_order.append(u)
>
>        for v in adj[u]:
>            indegree[v] -= 1
>            if indegree[v] == 0:
>                queue.append(v)
>
>    # Step 4: Check for cycles
>    if len(topological_order) == num_vertices:
>        return topological_order
>    else:
>        return [] # Or raise an error indicating a cycle
>```

>[!example]- JavaScript
>```javascript
>function kahnsTopologicalSort(numVertices, edges) {
>    // edges: array of arrays [[u, v], ...] representing directed edge from u to v
>
>    // Step 1: Compute indegrees and build adjacency list
>    const adj = new Map();
>    const indegree = new Map();
>
>    for (let i = 0; i < numVertices; i++) {
>        adj.set(i, []);
>        indegree.set(i, 0);
>    }
>
>    for (const [u, v] of edges) {
>        adj.get(u).push(v);
>        indegree.set(v, indegree.get(v) + 1);
>    }
>
>    // Step 2: Initialize the queue
>    const queue = [];
>    for (let i = 0; i < numVertices; i++) {
>        if (indegree.get(i) === 0) {
>            queue.push(i);
>        }
>    }
>
>    const topologicalOrder = [];
>
>    // Step 3: Process nodes
>    while (queue.length > 0) {
>        const u = queue.shift();
>        topologicalOrder.push(u);
>
>        for (const v of adj.get(u)) {
>            indegree.set(v, indegree.get(v) - 1);
>            if (indegree.get(v) === 0) {
>                queue.push(v);
>            }
>        }
>    }
>
>    // Step 4: Check for cycles
>    if (topologicalOrder.length === numVertices) {
>        return topologicalOrder;
>    } else {
>        return []; // Return empty array indicating a cycle
>    }
>}
>```

Kahn's algorithm is often favored in interviews because its logic is very direct and it clearly demonstrates the dependency-clearing nature of topological sorting.
