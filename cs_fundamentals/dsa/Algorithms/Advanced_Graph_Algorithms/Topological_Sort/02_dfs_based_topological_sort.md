## Topological Sort: DFS-Based Algorithm

An alternative to Kahn's algorithm for topological sorting is a clever approach using **Depth-First Search (DFS)**. This method is often more concise to implement recursively.

### The Core Idea
The logic behind the DFS-based approach relies on the "finish time" of each vertex in the DFS traversal. When we perform a DFS, a vertex is considered "finished" only after the recursive calls for all its descendants have completed.

In a Directed Acyclic Graph (DAG), the last vertex to "finish" must be a source vertex (or one of them). It has no outgoing edges that lead to unvisited nodes. Therefore, if we add vertices to the *front* of our sorted list as they finish, we will end up with a valid topological order. The vertex that finishes last will be the first in our list, and the vertex that finishes first (a leaf in the DFS tree) will be the last.

### The Algorithm
1.  **Initialization**:
    - Initialize an empty list, `topological_order`, which will store the result. A `deque` is efficient for adding to the front.
    - Initialize a `visited` set to keep track of visited nodes during the DFS traversal.

2.  **Main Loop**:
    - Iterate through all the vertices in the graph.
    - If a vertex has not been visited, call a recursive `dfs` helper function on it.

3.  **DFS Helper Function (`dfs(node)`)**:
    a. Mark the current `node` as visited.
    b. For each `neighbor` of the `node`:
       - If the `neighbor` has not been visited, recursively call `dfs(neighbor)`.
    c. **Crucial Step**: After the recursion for all neighbors has completed, the current `node` is "finished." Add it to the **front** of the `topological_order` list.

4.  **Termination**: After iterating through all vertices, the `topological_order` list will contain a valid sort.

### Cycle Detection
This DFS approach can also be adapted to detect cycles. To do this, you need to maintain a second set, often called `recursion_stack` or `visiting`.
- When you begin the DFS for a node, add it to `visiting`.
- When the DFS for that node finishes (after all its neighbors have been explored), remove it from `visiting`.
- If, during a traversal, you encounter a neighbor that is already in the `visiting` set, you have found a back edge, which indicates a cycle in the graph.

### Implementation

>[!example]- C++
>```cpp
>#include <vector>
>#include <unordered_map>
>#include <unordered_set>
>#include <deque>
>using namespace std;
>
>vector<int> dfsTopologicalSort(unordered_map<int, vector<int>>& graph) {
>    // graph: adjacency list {node: [neighbors...]}
>
>    deque<int> topologicalOrder;
>    unordered_set<int> visited;
>    unordered_set<int> path; // For cycle detection
>
>    bool hasCycle = false;
>
>    function<void(int)> dfs = [&](int node) {
>        visited.insert(node);
>        path.insert(node);
>
>        if (graph.find(node) != graph.end()) {
>            for (int neighbor : graph[node]) {
>                if (path.count(neighbor)) {
>                    // Cycle detected
>                    hasCycle = true;
>                    return;
>                }
>                if (!visited.count(neighbor)) {
>                    dfs(neighbor);
>                    if (hasCycle) return; // Propagate cycle detection
>                }
>            }
>        }
>
>        // Remove node from path after exploring its descendants
>        path.erase(node);
>        // Add node to the front of the list
>        topologicalOrder.push_front(node);
>    };
>
>    // Call DFS for all unvisited nodes
>    for (const auto& [node, _] : graph) {
>        if (!visited.count(node)) {
>            dfs(node);
>            if (hasCycle) {
>                return {}; // Cycle detected, return empty vector
>            }
>        }
>    }
>
>    return vector<int>(topologicalOrder.begin(), topologicalOrder.end());
>}
>```

>[!example]- Java
>```java
>import java.util.*;
>
>public class DFSTopologicalSort {
>    private static boolean hasCycle;
>
>    public static List<Integer> dfsTopologicalSort(Map<Integer, List<Integer>> graph) {
>        // graph: adjacency list {node: [neighbors...]}
>
>        Deque<Integer> topologicalOrder = new LinkedList<>();
>        Set<Integer> visited = new HashSet<>();
>        Set<Integer> path = new HashSet<>();
>
>        hasCycle = false;
>
>        for (int node : graph.keySet()) {
>            if (!visited.contains(node)) {
>                if (!dfs(node, graph, visited, path, topologicalOrder)) {
>                    return new ArrayList<>(); // Cycle detected
>                }
>            }
>        }
>
>        return new ArrayList<>(topologicalOrder);
>    }
>
>    private static boolean dfs(int node, Map<Integer, List<Integer>> graph,
>                               Set<Integer> visited, Set<Integer> path,
>                               Deque<Integer> topologicalOrder) {
>        visited.add(node);
>        path.add(node);
>
>        if (graph.containsKey(node)) {
>            for (int neighbor : graph.get(node)) {
>                if (path.contains(neighbor)) {
>                    // Cycle detected
>                    return false;
>                }
>                if (!visited.contains(neighbor)) {
>                    if (!dfs(neighbor, graph, visited, path, topologicalOrder)) {
>                        return false; // Propagate cycle detection
>                    }
>                }
>            }
>        }
>
>        // Remove node from path after exploring its descendants
>        path.remove(node);
>        // Add node to the front of the list
>        topologicalOrder.addFirst(node);
>        return true;
>    }
>}
>```

>[!example]- Python
>```python
>from collections import deque
>
>def dfs_topological_sort(graph):
>    """
>    Performs a topological sort using a DFS-based algorithm.
>    Args:
>      graph: An adjacency list {node: [neighbors...]}
>    """
>    topological_order = deque()
>    visited = set()
>
>    # We use a path set to detect cycles
>    path = set()
>
>    def dfs(node):
>        visited.add(node)
>        path.add(node)
>
>        for neighbor in graph.get(node, []):
>            if neighbor in path:
>                # Cycle detected
>                return False
>            if neighbor not in visited:
>                if not dfs(neighbor):
>                    return False # Propagate cycle detection
>
>        # Remove node from path after exploring its descendants
>        path.remove(node)
>        # Add node to the front of the list
>        topological_order.appendleft(node)
>        return True
>
>    # Call DFS for all unvisited nodes
>    for node in list(graph.keys()):
>        if node not in visited:
>            if not dfs(node):
>                return [] # Cycle detected, return empty list
>
>    return list(topological_order)
>
># Example:
># graph = {0: [1, 2], 1: [3], 2: [3], 3: []}
># dfs_topological_sort(graph) would return a valid sort like [0, 2, 1, 3] or [0, 1, 2, 3]
>```

>[!example]- JavaScript
>```javascript
>function dfsTopologicalSort(graph) {
>    // graph: adjacency list {node: [neighbors...]}
>
>    const topologicalOrder = [];
>    const visited = new Set();
>    const path = new Set(); // For cycle detection
>
>    function dfs(node) {
>        visited.add(node);
>        path.add(node);
>
>        const neighbors = graph[node] || [];
>        for (const neighbor of neighbors) {
>            if (path.has(neighbor)) {
>                // Cycle detected
>                return false;
>            }
>            if (!visited.has(neighbor)) {
>                if (!dfs(neighbor)) {
>                    return false; // Propagate cycle detection
>                }
>            }
>        }
>
>        // Remove node from path after exploring its descendants
>        path.delete(node);
>        // Add node to the front of the list
>        topologicalOrder.unshift(node);
>        return true;
>    }
>
>    // Call DFS for all unvisited nodes
>    for (const node in graph) {
>        if (!visited.has(parseInt(node))) {
>            if (!dfs(parseInt(node))) {
>                return []; // Cycle detected, return empty array
>            }
>        }
>    }
>
>    return topologicalOrder;
>}
>
>// Example:
>// const graph = {0: [1, 2], 1: [3], 2: [3], 3: []};
>// dfsTopologicalSort(graph) would return a valid sort like [0, 2, 1, 3] or [0, 1, 2, 3]
>```

This recursive approach is very elegant and powerful, combining both traversal and sorting in a single conceptual framework.
