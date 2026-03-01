## Depth-First Search (DFS) on Graphs

Depth-First Search (DFS) is a graph traversal algorithm that explores as far as possible along each branch before backtracking. It's a fundamental algorithm used as a building block for many other complex graph problems.

### Core Idea vs. Tree DFS
DFS on a graph operates on the same principle as DFS on a tree: go as deep as you can, then backtrack. It is most naturally implemented using recursion (which uses the implicit function call stack), but can also be implemented iteratively with an explicit stack.

Just like with BFS, the most important difference when applying DFS to a graph is handling **cycles**. To avoid infinite loops, you must keep track of nodes you have already visited. A `visited` hash set is essential.

The recursive algorithm is:
1.  Define a `dfs(node)` function.
2.  Inside the function, first mark the current `node` as visited.
3.  Process the `node`.
4.  For each `neighbor` of the `node`:
    -   If the `neighbor` has not been visited, recursively call `dfs(neighbor)`.

### When to Use DFS
While BFS is the clear choice for shortest path problems, DFS is often a more natural fit for problems related to:
- **Path Existence**: Finding if *any* path exists between two nodes.
- **Topological Sorting**: Ordering the nodes of a Directed Acyclic Graph (DAG).
- **Cycle Detection**: Detecting if a graph has cycles.
- **Connected Components**: Finding all the connected parts of a graph.
- **Backtracking Problems**: Many problems that can be modeled as a state-space search, like mazes or puzzles (e.g., Word Search), are solved with a backtracking approach, which is a form of DFS.

### Implementation (Recursive)
Here is a generic recursive DFS implementation.

>[!example]- C++
>```cpp
>#include <unordered_map>
>#include <unordered_set>
>#include <vector>
>#include <iostream>
>using namespace std;
>
>void traverseDFS(unordered_map<int, vector<int>>& graph, int startNode) {
>    unordered_set<int> visited;
>
>    // Helper function for recursive DFS
>    function<void(int)> dfs = [&](int node) {
>        // 1. Mark node as visited
>        visited.insert(node);
>        cout << "Visiting node: " << node << endl; // Process node
>
>        // 2. Recurse for all unvisited neighbors
>        if (graph.find(node) != graph.end()) {
>            for (int neighbor : graph[node]) {
>                if (visited.find(neighbor) == visited.end()) {
>                    dfs(neighbor);
>                }
>            }
>        }
>    };
>
>    // Start the traversal if the start_node is in the graph
>    if (graph.find(startNode) != graph.end()) {
>        dfs(startNode);
>    }
>}
>```

>[!example]- Java
>```java
>import java.util.*;
>
>public void traverseDFS(Map<Integer, List<Integer>> graph, int startNode) {
>    Set<Integer> visited = new HashSet<>();
>
>    dfsHelper(graph, startNode, visited);
>}
>
>private void dfsHelper(Map<Integer, List<Integer>> graph, int node, Set<Integer> visited) {
>    // 1. Mark node as visited
>    visited.add(node);
>    System.out.println("Visiting node: " + node); // Process node
>
>    // 2. Recurse for all unvisited neighbors
>    if (graph.containsKey(node)) {
>        for (int neighbor : graph.get(node)) {
>            if (!visited.contains(neighbor)) {
>                dfsHelper(graph, neighbor, visited);
>            }
>        }
>    }
>}
>```

>[!example]- Python
>```python
>def traverse_dfs(graph, start_node):
>    visited = set()
>
>    def dfs(node):
>        # 1. Mark node as visited
>        visited.add(node)
>        print(f"Visiting node: {node}") # Process node
>
>        # 2. Recurse for all unvisited neighbors
>        for neighbor in graph.get(node, []): # .get() avoids error if node has no outgoing edges
>            if neighbor not in visited:
>                dfs(neighbor)
>
>    # Start the traversal if the start_node is in the graph
>    if start_node in graph:
>        dfs(start_node)
>```

>[!example]- JavaScript
>```javascript
>function traverseDFS(graph, startNode) {
>    const visited = new Set();
>
>    function dfs(node) {
>        // 1. Mark node as visited
>        visited.add(node);
>        console.log(`Visiting node: ${node}`); // Process node
>
>        // 2. Recurse for all unvisited neighbors
>        const neighbors = graph.get(node) || [];
>        for (const neighbor of neighbors) {
>            if (!visited.has(neighbor)) {
>                dfs(neighbor);
>            }
>        }
>    }
>
>    // Start the traversal if the start_node is in the graph
>    if (graph.has(startNode)) {
>        dfs(startNode);
>    }
>}
>```

### Example: Finding Path Existence
A simple and common use of DFS is to check if a path exists between a `start` and `end` node.

>[!example]- C++
>```cpp
>#include <unordered_map>
>#include <unordered_set>
>#include <vector>
>using namespace std;
>
>bool canFindPathDFS(unordered_map<int, vector<int>>& graph, int start, int end) {
>    unordered_set<int> visited;
>
>    // Helper function for recursive DFS
>    function<bool(int)> dfs = [&](int node) -> bool {
>        // If we've reached the end, a path exists
>        if (node == end) {
>            return true;
>        }
>
>        visited.insert(node);
>
>        if (graph.find(node) != graph.end()) {
>            for (int neighbor : graph[node]) {
>                if (visited.find(neighbor) == visited.end()) {
>                    // If the recursive call finds the end, propagate true up
>                    if (dfs(neighbor)) {
>                        return true;
>                    }
>                }
>            }
>        }
>
>        // If no path was found from this node
>        return false;
>    };
>
>    return (graph.find(start) != graph.end()) ? dfs(start) : false;
>}
>```

>[!example]- Java
>```java
>import java.util.*;
>
>public boolean canFindPathDFS(Map<Integer, List<Integer>> graph, int start, int end) {
>    Set<Integer> visited = new HashSet<>();
>    return dfsPathHelper(graph, start, end, visited);
>}
>
>private boolean dfsPathHelper(Map<Integer, List<Integer>> graph, int node, int end, Set<Integer> visited) {
>    // If we've reached the end, a path exists
>    if (node == end) {
>        return true;
>    }
>
>    visited.add(node);
>
>    if (graph.containsKey(node)) {
>        for (int neighbor : graph.get(node)) {
>            if (!visited.contains(neighbor)) {
>                // If the recursive call finds the end, propagate true up
>                if (dfsPathHelper(graph, neighbor, end, visited)) {
>                    return true;
>                }
>            }
>        }
>    }
>
>    // If no path was found from this node
>    return false;
>}
>```

>[!example]- Python
>```python
>def can_find_path_dfs(graph, start, end):
>    visited = set()
>
>    def dfs(node):
>        # If we've reached the end, a path exists
>        if node == end:
>            return True
>
>        visited.add(node)
>
>        for neighbor in graph.get(node, []):
>            if neighbor not in visited:
>                # If the recursive call finds the end, propagate True up
>                if dfs(neighbor):
>                    return True
>
>        # If no path was found from this node
>        return False
>
>    return dfs(start) if start in graph else False
>```

>[!example]- JavaScript
>```javascript
>function canFindPathDFS(graph, start, end) {
>    const visited = new Set();
>
>    function dfs(node) {
>        // If we've reached the end, a path exists
>        if (node === end) {
>            return true;
>        }
>
>        visited.add(node);
>
>        const neighbors = graph.get(node) || [];
>        for (const neighbor of neighbors) {
>            if (!visited.has(neighbor)) {
>                // If the recursive call finds the end, propagate true up
>                if (dfs(neighbor)) {
>                    return true;
>                }
>            }
>        }
>
>        // If no path was found from this node
>        return false;
>    }
>
>    return graph.has(start) ? dfs(start) : false;
>}
>```

This is often simpler to write recursively than the equivalent BFS, which is why many programmers default to DFS when either traversal would work.
