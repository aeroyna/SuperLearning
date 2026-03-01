## Prim's Algorithm

Prim's Algorithm is the other classic greedy algorithm for finding a Minimum Spanning Tree (MST) in a connected, undirected, and weighted graph. While Kruskal's algorithm builds an MST by connecting components, Prim's algorithm builds it by growing a single tree, one edge at a time.

### The Core Idea
Prim's algorithm works by starting from an arbitrary vertex and growing the MST by adding the cheapest possible edge that connects a vertex in the growing tree to a vertex outside the tree. It is very similar in structure to Dijkstra's algorithm for shortest paths.

Like Dijkstra's, Prim's uses a **priority queue** (min-heap) to efficiently select the next edge to add.

### The Algorithm
1.  **Initialization**:
    - Choose an arbitrary `start_node`.
    - Initialize an `mst` list to store the edges of the final MST.
    - Initialize a `visited` set to keep track of vertices already included in the MST.
    - Initialize a min-priority queue. Add all edges starting from the `start_node` to the priority queue. The priority is the edge weight. Mark the `start_node` as visited.

2.  **Main Loop**:
    - While the priority queue is not empty and the MST is not yet complete (`mst` has fewer than `V-1` edges):
      a. Extract the edge with the minimum weight from the priority queue. Let this be `(weight, u, v)`, where `u` is a vertex already in the MST and `v` is a vertex not yet in the MST.
      b. If `v` has already been visited, discard this edge and continue (this prevents cycles).
      c. If `v` has not been visited:
         i. Mark `v` as visited.
         ii. Add the edge `(u, v, weight)` to the `mst` list.
         iii. Iterate through all the neighbors of `v`. For each neighbor `w`, if `w` has not been visited, add the edge `(v, w)` with its weight to the priority queue.

3.  **Termination**: The algorithm ends when the MST has `V-1` edges.

### Complexity
- **Time Complexity**: **O(E log V)** (or O(E log E)).
  - With a binary heap implementation for the priority queue, the complexity is dominated by the edge operations. In the worst case, every edge is added to the priority queue once. Each `heappop` is O(log E) and each `heappush` is O(log E). Since `log E` is O(log V) in a connected graph, the total time is O(E log V).
- **Space Complexity**: **O(V + E)** to store the graph, the visited set, and the priority queue.

### Kruskal's vs. Prim's
- **Prim's Algorithm** is generally faster for **dense graphs** (where E is close to V^2).
- **Kruskal's Algorithm** is generally faster for **sparse graphs** (where E is much smaller than V^2), because its runtime depends more on sorting the edges.
- Prim's resembles Dijkstra's, growing a tree from a single source. Kruskal's builds a "forest" of trees that eventually merge into one.

### Implementation

>[!example]- C++
>```cpp
>#include <vector>
>#include <queue>
>#include <unordered_map>
>#include <unordered_set>
>using namespace std;
>
>pair<vector<tuple<int, int, int>>, int> primsAlgorithm(
>    unordered_map<int, vector<pair<int, int>>>& graph, int startNode) {
>    // graph: adjacency list {node: [(neighbor, weight), ...]}
>
>    vector<tuple<int, int, int>> mst;
>    int totalWeight = 0;
>    unordered_set<int> visited;
>    visited.insert(startNode);
>
>    // Priority queue stores (weight, u, v) for the edge from u to v
>    priority_queue<tuple<int, int, int>, vector<tuple<int, int, int>>,
>                   greater<tuple<int, int, int>>> pq;
>
>    for (const auto& [neighbor, weight] : graph[startNode]) {
>        pq.push({weight, startNode, neighbor});
>    }
>
>    while (!pq.empty() && mst.size() < graph.size() - 1) {
>        auto [weight, u, v] = pq.top();
>        pq.pop();
>
>        // If the destination node is already in our MST, skip
>        if (visited.count(v)) {
>            continue;
>        }
>
>        // Add the new node and edge to the MST
>        visited.insert(v);
>        mst.push_back({u, v, weight});
>        totalWeight += weight;
>
>        // Add all outgoing edges from the new node to the PQ
>        if (graph.count(v)) {
>            for (const auto& [nextNeighbor, nextWeight] : graph[v]) {
>                if (!visited.count(nextNeighbor)) {
>                    pq.push({nextWeight, v, nextNeighbor});
>                }
>            }
>        }
>    }
>
>    // Check if a valid MST was formed
>    if (visited.size() == graph.size()) {
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
>public class PrimsAlgorithm {
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
>    public static Result primsAlgorithm(Map<Integer, List<int[]>> graph, int startNode) {
>        // graph: adjacency list {node: [[neighbor, weight], ...]}
>
>        List<int[]> mst = new ArrayList<>();
>        int totalWeight = 0;
>        Set<Integer> visited = new HashSet<>();
>        visited.add(startNode);
>
>        // Priority queue stores [weight, u, v] for the edge from u to v
>        PriorityQueue<int[]> pq = new PriorityQueue<>(Comparator.comparingInt(a -> a[0]));
>
>        if (graph.containsKey(startNode)) {
>            for (int[] edge : graph.get(startNode)) {
>                int neighbor = edge[0];
>                int weight = edge[1];
>                pq.offer(new int[]{weight, startNode, neighbor});
>            }
>        }
>
>        while (!pq.isEmpty() && mst.size() < graph.size() - 1) {
>            int[] current = pq.poll();
>            int weight = current[0];
>            int u = current[1];
>            int v = current[2];
>
>            // If the destination node is already in our MST, skip
>            if (visited.contains(v)) {
>                continue;
>            }
>
>            // Add the new node and edge to the MST
>            visited.add(v);
>            mst.add(new int[]{u, v, weight});
>            totalWeight += weight;
>
>            // Add all outgoing edges from the new node to the PQ
>            if (graph.containsKey(v)) {
>                for (int[] edge : graph.get(v)) {
>                    int nextNeighbor = edge[0];
>                    int nextWeight = edge[1];
>                    if (!visited.contains(nextNeighbor)) {
>                        pq.offer(new int[]{nextWeight, v, nextNeighbor});
>                    }
>                }
>            }
>        }
>
>        // Check if a valid MST was formed
>        if (visited.size() == graph.size()) {
>            return new Result(mst, totalWeight);
>        } else {
>            return new Result(null, Integer.MAX_VALUE); // Graph is not connected
>        }
>    }
>}
>```

>[!example]- Python
>```python
>import heapq
>
>def prims_algorithm(graph, start_node):
>    """
>    Finds the MST of a graph using Prim's algorithm.
>    Args:
>      graph: An adjacency list {node: [(neighbor, weight), ...]}
>    """
>
>    mst = []
>    total_weight = 0
>    visited = {start_node}
>
>    # Priority queue stores (weight, u, v) for the edge from u to v
>    pq = []
>    for neighbor, weight in graph.get(start_node, []):
>        heapq.heappush(pq, (weight, start_node, neighbor))
>
>    while pq and len(mst) < len(graph) - 1:
>        weight, u, v = heapq.heappop(pq)
>
>        # If the destination node is already in our MST, skip
>        if v in visited:
>            continue
>
>        # Add the new node and edge to the MST
>        visited.add(v)
>        mst.append((u, v, weight))
>        total_weight += weight
>
>        # Add all outgoing edges from the new node to the PQ
>        for next_neighbor, next_weight in graph.get(v, []):
>            if next_neighbor not in visited:
>                heapq.heappush(pq, (next_weight, v, next_neighbor))
>
>    # Check if a valid MST was formed
>    if len(visited) == len(graph):
>        return mst, total_weight
>    else:
>        return None, float('inf') # Graph is not connected
>```

>[!example]- JavaScript
>```javascript
>class MinHeap {
>    constructor() {
>        this.heap = [];
>    }
>
>    push(item) {
>        this.heap.push(item);
>        this._bubbleUp(this.heap.length - 1);
>    }
>
>    pop() {
>        if (this.heap.length === 0) return null;
>        if (this.heap.length === 1) return this.heap.pop();
>
>        const min = this.heap[0];
>        this.heap[0] = this.heap.pop();
>        this._bubbleDown(0);
>        return min;
>    }
>
>    _bubbleUp(index) {
>        while (index > 0) {
>            const parentIndex = Math.floor((index - 1) / 2);
>            if (this.heap[index][0] >= this.heap[parentIndex][0]) break;
>            [this.heap[index], this.heap[parentIndex]] = [this.heap[parentIndex], this.heap[index]];
>            index = parentIndex;
>        }
>    }
>
>    _bubbleDown(index) {
>        while (true) {
>            const leftChild = 2 * index + 1;
>            const rightChild = 2 * index + 2;
>            let smallest = index;
>
>            if (leftChild < this.heap.length && this.heap[leftChild][0] < this.heap[smallest][0]) {
>                smallest = leftChild;
>            }
>            if (rightChild < this.heap.length && this.heap[rightChild][0] < this.heap[smallest][0]) {
>                smallest = rightChild;
>            }
>            if (smallest === index) break;
>
>            [this.heap[index], this.heap[smallest]] = [this.heap[smallest], this.heap[index]];
>            index = smallest;
>        }
>    }
>
>    isEmpty() {
>        return this.heap.length === 0;
>    }
>}
>
>function primsAlgorithm(graph, startNode) {
>    // graph: adjacency list {node: [[neighbor, weight], ...]}
>
>    const mst = [];
>    let totalWeight = 0;
>    const visited = new Set([startNode]);
>
>    // Priority queue stores [weight, u, v] for the edge from u to v
>    const pq = new MinHeap();
>
>    const neighbors = graph[startNode] || [];
>    for (const [neighbor, weight] of neighbors) {
>        pq.push([weight, startNode, neighbor]);
>    }
>
>    while (!pq.isEmpty() && mst.length < Object.keys(graph).length - 1) {
>        const [weight, u, v] = pq.pop();
>
>        // If the destination node is already in our MST, skip
>        if (visited.has(v)) {
>            continue;
>        }
>
>        // Add the new node and edge to the MST
>        visited.add(v);
>        mst.push([u, v, weight]);
>        totalWeight += weight;
>
>        // Add all outgoing edges from the new node to the PQ
>        const nextNeighbors = graph[v] || [];
>        for (const [nextNeighbor, nextWeight] of nextNeighbors) {
>            if (!visited.has(nextNeighbor)) {
>                pq.push([nextWeight, v, nextNeighbor]);
>            }
>        }
>    }
>
>    // Check if a valid MST was formed
>    if (visited.size === Object.keys(graph).length) {
>        return { mst, totalWeight };
>    } else {
>        return { mst: null, totalWeight: Infinity }; // Graph is not connected
>    }
>}
>```
