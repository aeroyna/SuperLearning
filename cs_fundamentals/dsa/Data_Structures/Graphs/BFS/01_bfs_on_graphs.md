## Breadth-First Search (BFS) on Graphs

Breadth-First Search (BFS) is a graph traversal algorithm that explores nodes "layer by layer." It starts at a source node and explores all of its immediate neighbors first, before moving on to the next level of neighbors. This property makes BFS the go-to algorithm for finding the **shortest path in an unweighted graph**.

### Core Idea vs. Tree BFS
BFS on a graph is very similar to BFS on a tree. It uses a **queue** to manage the order of nodes to visit, ensuring a level-by-level traversal.

However, there is one critical difference: **graphs can have cycles**. To prevent getting trapped in an infinite loop (e.g., A -> B -> A -> B...), you **must** keep track of nodes that have already been visited. A `visited` hash set is used for this purpose.

The algorithm is as follows:
1. Initialize a `queue` and add the `start_node`.
2. Initialize a `visited` set and add the `start_node`.
3. While the queue is not empty:
   a. Dequeue a `node`.
   b. Process the `node`.
   c. For each `neighbor` of the `node`:
      i. If the `neighbor` has not been visited, add it to the `visited` set and enqueue it.

### Main Application: Shortest Path
If a problem asks for the "shortest path," "fewest steps," "minimum moves," or "closest" distance in an unweighted graph (including implicit grids), BFS should be your immediate thought.

Because BFS explores layer by layer, the first time it reaches the target node, it is guaranteed to have done so via a shortest path.

### Implementation
Here is a generic BFS implementation for finding the shortest path from a `start_node` to an `end_node`.

> [!example]- C++
> ```cpp
> #include <queue>
> #include <unordered_map>
> #include <unordered_set>
> #include <vector>
> using namespace std;
>
> int shortest_path_bfs(unordered_map<int, vector<int>>& graph, int start_node, int end_node) {
>     // Check for invalid start/end nodes
>     if (graph.find(start_node) == graph.end() || graph.find(end_node) == graph.end()) {
>         return -1;
>     }
>     if (start_node == end_node) {
>         return 0;
>     }
>
>     // Queue stores pairs of (node, distance_from_start)
>     queue<pair<int, int>> q;
>     q.push({start_node, 0});
>     unordered_set<int> visited;
>     visited.insert(start_node);
>
>     while (!q.empty()) {
>         auto [current_node, distance] = q.front();
>         q.pop();
>
>         for (int neighbor : graph[current_node]) {
>             if (neighbor == end_node) {
>                 return distance + 1;  // Path found
>             }
>
>             if (visited.find(neighbor) == visited.end()) {
>                 visited.insert(neighbor);
>                 q.push({neighbor, distance + 1});
>             }
>         }
>     }
>
>     return -1;  // Path not found
> }
> ```

> [!example]- Java
> ```java
> import java.util.*;
>
> public class BFS {
>     public static int shortestPathBFS(Map<Integer, List<Integer>> graph, int startNode, int endNode) {
>         // Check for invalid start/end nodes
>         if (!graph.containsKey(startNode) || !graph.containsKey(endNode)) {
>             return -1;
>         }
>         if (startNode == endNode) {
>             return 0;
>         }
>
>         // Queue stores arrays of [node, distance_from_start]
>         Queue<int[]> queue = new LinkedList<>();
>         queue.offer(new int[]{startNode, 0});
>         Set<Integer> visited = new HashSet<>();
>         visited.add(startNode);
>
>         while (!queue.isEmpty()) {
>             int[] current = queue.poll();
>             int currentNode = current[0];
>             int distance = current[1];
>
>             for (int neighbor : graph.get(currentNode)) {
>                 if (neighbor == endNode) {
>                     return distance + 1;  // Path found
>                 }
>
>                 if (!visited.contains(neighbor)) {
>                     visited.add(neighbor);
>                     queue.offer(new int[]{neighbor, distance + 1});
>                 }
>             }
>         }
>
>         return -1;  // Path not found
>     }
> }
> ```

> [!example]- Python
> ```python
> from collections import deque
>
> def shortest_path_bfs(graph, start_node, end_node):
>     # Check for invalid start/end nodes
>     if start_node not in graph or end_node not in graph:
>         return -1
>     if start_node == end_node:
>         return 0
>
>     # Queue stores tuples of (node, distance_from_start)
>     queue = deque([(start_node, 0)])
>     visited = {start_node}
>
>     while queue:
>         current_node, distance = queue.popleft()
>
>         for neighbor in graph[current_node]:
>             if neighbor == end_node:
>                 return distance + 1  # Path found
>
>             if neighbor not in visited:
>                 visited.add(neighbor)
>                 queue.append((neighbor, distance + 1))
>
>     return -1 # Path not found
> ```

> [!example]- JavaScript
> ```javascript
> function shortestPathBFS(graph, startNode, endNode) {
>     // Check for invalid start/end nodes
>     if (!(startNode in graph) || !(endNode in graph)) {
>         return -1;
>     }
>     if (startNode === endNode) {
>         return 0;
>     }
>
>     // Queue stores arrays of [node, distance_from_start]
>     const queue = [[startNode, 0]];
>     const visited = new Set([startNode]);
>
>     while (queue.length > 0) {
>         const [currentNode, distance] = queue.shift();
>
>         for (const neighbor of graph[currentNode]) {
>             if (neighbor === endNode) {
>                 return distance + 1;  // Path found
>             }
>
>             if (!visited.has(neighbor)) {
>                 visited.add(neighbor);
>                 queue.push([neighbor, distance + 1]);
>             }
>         }
>     }
>
>     return -1;  // Path not found
> }
> ```

This template can be adapted for many shortest-path problems. For example, on a grid, the "node" would be a `(row, col)` tuple.
