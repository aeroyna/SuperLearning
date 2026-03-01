# Graph Basics

Graphs model relationships between entities. Understanding graph representations, their trade-offs, and fundamental properties is essential before tackling graph algorithms.

## Overview

A graph G = (V, E) consists of:
- **V (Vertices/Nodes)**: The entities
- **E (Edges)**: Connections between entities

## Topics

- [11.1.1 Graph Representations](01_graph_representations.md)
- [11.1.2 Implicit Graphs](02_implicit_graphs.md)

## Graph Types

### By Edge Direction

```
Undirected:     Directed (Digraph):
  A --- B         A --→ B
  |     |         ↑     |
  C --- D         C ←-- D
```

**Undirected**: Edge (A, B) implies edge (B, A)
**Directed**: Edge A→B does not imply B→A

### By Edge Weight

**Unweighted**: All edges have equal cost
**Weighted**: Edges have associated costs

### Special Types

- **DAG (Directed Acyclic Graph)**: No cycles, enables topological sort
- **Tree**: Connected graph with n-1 edges (no cycles)
- **Bipartite**: Vertices split into two sets, edges only between sets
- **Complete**: Edge between every pair of vertices

## Graph Representations

### Adjacency List (Most Common)

>[!example]- C++
>```cpp
>// Using map of vectors
>unordered_map<string, vector<string>> graph = {
>    {"A", {"B", "C"}},
>    {"B", {"A", "D"}},
>    // ...
>};
>
>// Using vector of vectors (0 to n-1)
>vector<vector<int>> graph = {
>    {1, 2},    // Node 0
>    {0, 3},    // Node 1
>    {0, 3},    // Node 2
>    {1, 2}     // Node 3
>};
>```

>[!example]- Java
>```java
>// Using Map of Lists
>Map<String, List<String>> graph = new HashMap<>();
>graph.put("A", Arrays.asList("B", "C"));
>
>// Using List of Lists (0 to n-1)
>List<List<Integer>> graph = new ArrayList<>();
>graph.add(Arrays.asList(1, 2)); // Node 0
>```

>[!example]- Python
>```python
># Using dictionary of lists
>graph = {
>    'A': ['B', 'C'],
>    'B': ['A', 'D'],
>    'C': ['A', 'D'],
>    'D': ['B', 'C']
>}
>
># Using list of lists (when vertices are 0 to n-1)
>graph = [
>    [1, 2],    # Node 0's neighbors
>    [0, 3],    # Node 1's neighbors
>    [0, 3],    # Node 2's neighbors
>    [1, 2]     # Node 3's neighbors
>]
>
># With weights
>graph = {
>    'A': [('B', 5), ('C', 3)],
>    'B': [('A', 5), ('D', 2)]
>}
>```

>[!example]- JavaScript
>```javascript
>// Using Map of arrays
>const graph = new Map([
>    ['A', ['B', 'C']],
>    ['B', ['A', 'D']]
>]);
>
>// Using array of arrays
>const graph = [
>    [1, 2],    // Node 0
>    [0, 3],    // Node 1
>    [0, 3],    // Node 2
>    [1, 2]     // Node 3
>];
>```

### Adjacency Matrix

>[!example]- C++
>```cpp
>vector<vector<int>> matrix = {
>    {0, 1, 1, 0},
>    {1, 0, 0, 1},
>    {1, 0, 0, 1},
>    {0, 1, 1, 0}
>};
>```

>[!example]- Java
>```java
>int[][] matrix = {
>    {0, 1, 1, 0},
>    {1, 0, 0, 1},
>    {1, 0, 0, 1},
>    {0, 1, 1, 0}
>};
>```

>[!example]- Python
>```python
># 4 vertices (0-indexed)
>#     0  1  2  3
>matrix = [
>    [0, 1, 1, 0],  # 0
>    [1, 0, 0, 1],  # 1
>    [1, 0, 0, 1],  # 2
>    [0, 1, 1, 0]   # 3
>]
>
># matrix[i][j] = 1 means edge from i to j
># For weighted: matrix[i][j] = weight (0 or inf for no edge)
>```

>[!example]- JavaScript
>```javascript
>const matrix = [
>    [0, 1, 1, 0],
>    [1, 0, 0, 1],
>    [1, 0, 0, 1],
>    [0, 1, 1, 0]
>];
>```

### Edge List

```python
edges = [
    ('A', 'B'),
    ('A', 'C'),
    ('B', 'D'),
    ('C', 'D')
]

# With weights
edges = [
    ('A', 'B', 5),
    ('A', 'C', 3),
    ('B', 'D', 2)
]
```

## Representation Trade-offs

| Operation | Adjacency List | Adjacency Matrix |
|-----------|---------------|------------------|
| Space | O(V + E) | O(V²) |
| Check edge exists | O(degree) | O(1) |
| Get all neighbors | O(degree) | O(V) |
| Add edge | O(1) | O(1) |
| Remove edge | O(degree) | O(1) |

**Decision rule**:
- **Sparse graph** (E << V²): Use adjacency list
- **Dense graph** (E ≈ V²): Matrix may be appropriate
- **Need O(1) edge lookup**: Matrix
- **Memory constrained**: List

## Building Graphs from Input

### Edge List to Adjacency List

>[!example]- C++
>```cpp
>unordered_map<int, vector<int>> buildGraph(vector<pair<int, int>>& edges, bool directed) {
>    unordered_map<int, vector<int>> graph;
>    for (const auto& edge : edges) {
>        graph[edge.first].push_back(edge.second);
>        if (!directed) {
>            graph[edge.second].push_back(edge.first);
>        }
>    }
>    return graph;
>}
>```

>[!example]- Java
>```java
>public Map<Integer, List<Integer>> buildGraph(int[][] edges, boolean directed) {
>    Map<Integer, List<Integer>> graph = new HashMap<>();
>    for (int[] edge : edges) {
>        int u = edge[0], v = edge[1];
>        graph.computeIfAbsent(u, k -> new ArrayList<>()).add(v);
>        if (!directed) {
>            graph.computeIfAbsent(v, k -> new ArrayList<>()).add(u);
>        }
>    }
>    return graph;
>}
>```

>[!example]- Python
>```python
>from collections import defaultdict
>
>def build_graph(edges, directed=False):
>    graph = defaultdict(list)
>    for u, v in edges:
>        graph[u].append(v)
>        if not directed:
>            graph[v].append(u)
>    return graph
>```

>[!example]- JavaScript
>```javascript
>function buildGraph(edges, directed = false) {
>    const graph = new Map();
>    for (const [u, v] of edges) {
>        if (!graph.has(u)) graph.set(u, []);
>        graph.get(u).push(v);
>        if (!directed) {
>            if (!graph.has(v)) graph.set(v, []);
>            graph.get(v).push(u);
>        }
>    }
>    return graph;
>}
>```

### Matrix to Adjacency List

```python
def matrix_to_adj_list(matrix):
    n = len(matrix)
    graph = defaultdict(list)
    for i in range(n):
        for j in range(n):
            if matrix[i][j]:  # Edge exists
                graph[i].append(j)
    return graph
```

## Implicit Graphs

Many problems have implicit graph structure without explicit representation:

### Grid as Graph

```python
def grid_neighbors(grid, r, c):
    """4-directional neighbors"""
    directions = [(0, 1), (0, -1), (1, 0), (-1, 0)]
    rows, cols = len(grid), len(grid[0])
    neighbors = []
    for dr, dc in directions:
        nr, nc = r + dr, c + dc
        if 0 <= nr < rows and 0 <= nc < cols:
            if grid[nr][nc] != '#':  # Not blocked
                neighbors.append((nr, nc))
    return neighbors
```

### State Space as Graph

```
Word Ladder: Each word is a node, edge if one letter different
Puzzle: Each state is a node, edge if one move connects states
```

## Key Graph Properties

### Degree

```python
def in_degree(graph, node):
    """For directed graph: count of edges into node"""
    count = 0
    for neighbors in graph.values():
        count += neighbors.count(node)
    return count

def out_degree(graph, node):
    """Edges out of node"""
    return len(graph[node])
```

### Connectivity

- **Connected** (undirected): Path exists between any two vertices
- **Strongly connected** (directed): Path in both directions between any two vertices
- **Weakly connected** (directed): Connected if edges were undirected

### Cycle Detection

```python
def has_cycle_undirected(graph):
    visited = set()

    def dfs(node, parent):
        visited.add(node)
        for neighbor in graph[node]:
            if neighbor not in visited:
                if dfs(neighbor, node):
                    return True
            elif neighbor != parent:
                return True  # Back edge to non-parent
        return False

    for node in graph:
        if node not in visited:
            if dfs(node, None):
                return True
    return False
```

## Common Pitfalls

1. **Forgetting bidirectional edges**: Undirected needs edges in both directions
2. **Missing isolated nodes**: Nodes with no edges might not appear in edge list
3. **Self-loops and multi-edges**: Clarify if graph allows these
4. **1-indexed vs 0-indexed**: Common source of off-by-one errors

## Key Interview Problems

| Problem | Concept | Difficulty | LeetCode Link |
| --------- | --------- | ------------ | --- |
| Clone Graph | Traversal + copying | Medium | [Link](https://leetcode.com/problems/clone-graph/) |
| Course Schedule | Cycle detection | Medium | [Link](https://leetcode.com/problems/course-schedule/) |
| Number of Islands | Grid as graph | Medium | [Link](https://leetcode.com/problems/number-of-islands/) |
| Graph Valid Tree | Cycle + connectivity | Medium | [Link](https://leetcode.com/problems/graph-valid-tree/) |
| Redundant Connection | Find cycle edge | Medium | [Link](https://leetcode.com/problems/redundant-connection/) |
