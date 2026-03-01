## Dijkstra's Algorithm

Dijkstra's Algorithm is a classic and widely used greedy algorithm for solving the **Single-Source Shortest Path (SSSP)** problem on a **weighted graph with non-negative edge weights**. It finds the shortest path from a given source node to all other nodes in the graph.

### The Core Idea
Dijkstra's works by maintaining a set of visited nodes and a data structure that stores the "tentative" shortest distance from the source to every other node. It greedily and repeatedly selects the unvisited node with the smallest known distance and explores its neighbors.

The algorithm uses a **priority queue** (min-heap) to efficiently select the unvisited node with the smallest distance at each step.

### The Algorithm
1.  **Initialization**:
    - Create a `distances` array to store the shortest distance from the `source` to every other node. Initialize all distances to infinity, except for the `source` node, which is 0.
    - Create a `visited` set to keep track of nodes for which we have already found the shortest path.
    - Initialize a min-priority queue and add the `source` node to it, with a priority of 0 (e.g., `(distance, node)`).

2.  **Main Loop**:
    - While the priority queue is not empty:
      a. Extract the node with the minimum distance from the priority queue. Let this be `(current_dist, current_node)`.
      b. If `current_node` has already been visited, skip it.
      c. Mark `current_node` as visited.
      d. For each `neighbor` of `current_node`:
         i. Calculate the distance to this neighbor through the current node: `new_dist = current_dist + weight_of_edge(current_node, neighbor)`.
         ii. If `new_dist` is smaller than the known distance to the `neighbor` (`distances[neighbor]`), it means we have found a new, shorter path.
         iii. Update `distances[neighbor] = new_dist` and add the `(new_dist, neighbor)` to the priority queue.

3.  **Termination**: The loop ends when the priority queue is empty. The `distances` array now holds the shortest path distances from the source to all other nodes.

### Implementation Notes
- **Priority Queue**: A min-heap is essential for an efficient implementation, giving the algorithm its O(E log V) complexity. The priority queue stores tuples of `(distance, vertex)`.
- **Non-Negative Weights**: Dijkstra's greedy approach fails if there are negative edge weights. The algorithm assumes that once a node is visited, its shortest path is finalized. A negative edge could violate this assumption by creating a shorter path to an already visited node. For graphs with negative edges, the Bellman-Ford algorithm must be used.

### Implementation

>[!example]- C++
>```cpp
>#include <vector>
>#include <queue>
>#include <unordered_map>
>#include <limits>
>using namespace std;
>
>unordered_map<int, int> dijkstra(unordered_map<int, vector<pair<int, int>>>& graph, int start_node) {
>    // graph: adjacency list {node: [(neighbor, weight), ...]}
>
>    unordered_map<int, int> distances;
>
>    // Initialize all distances to infinity
>    for (const auto& [node, _] : graph) {
>        distances[node] = numeric_limits<int>::max();
>    }
>    distances[start_node] = 0;
>
>    // Min-heap priority queue: (distance, node)
>    priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>>> pq;
>    pq.push({0, start_node});
>
>    while (!pq.empty()) {
>        auto [current_dist, current_node] = pq.top();
>        pq.pop();
>
>        // Skip if we've already found a shorter path
>        if (current_dist > distances[current_node]) {
>            continue;
>        }
>
>        // Explore neighbors
>        if (graph.find(current_node) != graph.end()) {
>            for (const auto& [neighbor, weight] : graph[current_node]) {
>                int distance = current_dist + weight;
>
>                // If we found a shorter path
>                if (distance < distances[neighbor]) {
>                    distances[neighbor] = distance;
>                    pq.push({distance, neighbor});
>                }
>            }
>        }
>    }
>
>    return distances;
>}
>```

>[!example]- Java
>```java
>import java.util.*;
>
>public class Dijkstra {
>    public static Map<Integer, Integer> dijkstra(Map<Integer, List<int[]>> graph, int startNode) {
>        // graph: adjacency list {node: [[neighbor, weight], ...]}
>
>        Map<Integer, Integer> distances = new HashMap<>();
>
>        // Initialize all distances to infinity
>        for (int node : graph.keySet()) {
>            distances.put(node, Integer.MAX_VALUE);
>        }
>        distances.put(startNode, 0);
>
>        // Min-heap priority queue: [distance, node]
>        PriorityQueue<int[]> pq = new PriorityQueue<>(Comparator.comparingInt(a -> a[0]));
>        pq.offer(new int[]{0, startNode});
>
>        while (!pq.isEmpty()) {
>            int[] current = pq.poll();
>            int currentDist = current[0];
>            int currentNode = current[1];
>
>            // Skip if we've already found a shorter path
>            if (currentDist > distances.get(currentNode)) {
>                continue;
>            }
>
>            // Explore neighbors
>            if (graph.containsKey(currentNode)) {
>                for (int[] edge : graph.get(currentNode)) {
>                    int neighbor = edge[0];
>                    int weight = edge[1];
>                    int distance = currentDist + weight;
>
>                    // If we found a shorter path
>                    if (distance < distances.get(neighbor)) {
>                        distances.put(neighbor, distance);
>                        pq.offer(new int[]{distance, neighbor});
>                    }
>                }
>            }
>        }
>
>        return distances;
>    }
>}
>```

>[!example]- Python
>```python
>import heapq
>
>def dijkstra(graph, start_node):
>    # graph should be an adjacency list: {node: [(neighbor, weight), ...]}
>
>    # Initialize distances to all nodes as infinity
>    distances = {node: float('inf') for node in graph}
>    # The distance to the start node is 0
>    distances[start_node] = 0
>
>    # Priority queue stores (distance, node)
>    pq = [(0, start_node)]
>
>    visited = set()
>
>    while pq:
>        # Get the node with the smallest distance
>        current_dist, current_node = heapq.heappop(pq)
>
>        # If we've already found a shorter path to this node, skip
>        if current_dist > distances[current_node]:
>            continue
>
>        # Explore neighbors
>        for neighbor, weight in graph.get(current_node, []):
>            distance = current_dist + weight
>
>            # If we found a shorter path to the neighbor
>            if distance < distances[neighbor]:
>                distances[neighbor] = distance
>                heapq.heappush(pq, (distance, neighbor))
>
>    return distances
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
>function dijkstra(graph, startNode) {
>    // graph: adjacency list {node: [[neighbor, weight], ...]}
>
>    const distances = {};
>
>    // Initialize all distances to infinity
>    for (const node in graph) {
>        distances[node] = Infinity;
>    }
>    distances[startNode] = 0;
>
>    // Min-heap priority queue: [distance, node]
>    const pq = new MinHeap();
>    pq.push([0, startNode]);
>
>    while (!pq.isEmpty()) {
>        const [currentDist, currentNode] = pq.pop();
>
>        // Skip if we've already found a shorter path
>        if (currentDist > distances[currentNode]) {
>            continue;
>        }
>
>        // Explore neighbors
>        const neighbors = graph[currentNode] || [];
>        for (const [neighbor, weight] of neighbors) {
>            const distance = currentDist + weight;
>
>            // If we found a shorter path
>            if (distance < distances[neighbor]) {
>                distances[neighbor] = distance;
>                pq.push([distance, neighbor]);
>            }
>        }
>    }
>
>    return distances;
>}
>```

This implementation finds the shortest distance from `start_node` to all other reachable nodes in the graph.
