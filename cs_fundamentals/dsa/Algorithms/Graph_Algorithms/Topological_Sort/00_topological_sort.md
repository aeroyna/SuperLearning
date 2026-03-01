# Topological Sort

Topological sort orders vertices of a Directed Acyclic Graph (DAG) such that for every edge u→v, u comes before v. It's essential for dependency resolution, build systems, and scheduling problems.

## Overview

A topological ordering exists if and only if the graph is a DAG (no cycles). A DAG may have multiple valid orderings.

Two approaches:
- **Kahn's Algorithm (BFS)**: Process vertices with zero in-degree
- **DFS-based**: Post-order traversal, reverse the result

## Topics

- [20.1 Kahn's Algorithm (BFS)](01_kahns_algorithm.md)
- [20.2 DFS-Based Topological Sort](02_dfs_topological_sort.md)

## Kahn's Algorithm (BFS)

Process vertices in order of zero in-degree.

>[!example]- C++
>```cpp
>vector<int> topologicalSortKahn(int n, vector<pair<int, int>>& edges) {
>    vector<vector<int>> graph(n);
>    vector<int> inDegree(n, 0);
>    
>    for (const auto& [u, v] : edges) {
>        graph[u].push_back(v);
>        inDegree[v]++;
>    }
>    
>    queue<int> q;
>    for (int i = 0; i < n; i++) {
>        if (inDegree[i] == 0) q.push(i);
>    }
>    
>    vector<int> result;
>    while (!q.empty()) {
>        int node = q.front();
>        q.pop();
>        result.push_back(node);
>        
>        for (int neighbor : graph[node]) {
>            inDegree[neighbor]--;
>            if (inDegree[neighbor] == 0) {
>                q.push(neighbor);
>            }
>        }
>    }
>    
>    if (result.size() != n) return {}; // Cycle detected
>    return result;
>}
>```

>[!example]- Java
>```java
>public int[] topologicalSortKahn(int n, int[][] edges) {
>    List<List<Integer>> graph = new ArrayList<>();
>    for (int i = 0; i < n; i++) graph.add(new ArrayList<>());
>    int[] inDegree = new int[n];
>    
>    for (int[] edge : edges) {
>        graph.get(edge[0]).add(edge[1]);
>        inDegree[edge[1]]++;
>    }
>    
>    Queue<Integer> queue = new LinkedList<>();
>    for (int i = 0; i < n; i++) {
>        if (inDegree[i] == 0) queue.offer(i);
>    }
>    
>    int[] result = new int[n];
>    int index = 0;
>    while (!queue.isEmpty()) {
>        int node = queue.poll();
>        result[index++] = node;
>        
>        for (int neighbor : graph.get(node)) {
>            inDegree[neighbor]--;
>            if (inDegree[neighbor] == 0) {
>                queue.offer(neighbor);
>            }
>        }
>    }
>    
>    if (index != n) return new int[0]; // Cycle detected
>    return result;
>}
>```

>[!example]- Python
>```python
>from collections import deque
>
>def topological_sort_kahn(n, edges):
>    """
>    n: number of vertices (0 to n-1)
>    edges: list of (u, v) meaning u must come before v
>    Returns: topological ordering, or empty list if cycle exists
>    """
>    graph = [[] for _ in range(n)]
>    in_degree = [0] * n
>
>    for u, v in edges:
>        graph[u].append(v)
>        in_degree[v] += 1
>
>    # Start with all zero in-degree vertices
>    queue = deque([i for i in range(n) if in_degree[i] == 0])
>    result = []
>
>    while queue:
>        node = queue.popleft()
>        result.append(node)
>
>        for neighbor in graph[node]:
>            in_degree[neighbor] -= 1
>            if in_degree[neighbor] == 0:
>                queue.append(neighbor)
>
>    # If not all vertices processed, cycle exists
>    if len(result) != n:
>        return []  # Cycle detected
>
>    return result
>```

>[!example]- JavaScript
>```javascript
>function topologicalSortKahn(n, edges) {
>    const graph = Array.from({ length: n }, () => []);
>    const inDegree = new Array(n).fill(0);
>    
>    for (const [u, v] of edges) {
>        graph[u].push(v);
>        inDegree[v]++;
>    }
>    
>    const queue = [];
>    for (let i = 0; i < n; i++) {
>        if (inDegree[i] === 0) queue.push(i);
>    }
>    
>    const result = [];
>    let head = 0; // Using index for queue efficiency
>    
>    while (head < queue.length) {
>        const node = queue[head++];
>        result.push(node);
>        
>        for (const neighbor of graph[node]) {
>            inDegree[neighbor]--;
>            if (inDegree[neighbor] === 0) {
>                queue.push(neighbor);
>            }
>        }
>    }
>    
>    if (result.length !== n) return []; // Cycle detected
>    return result;
>}
>```

### Why It Works

Vertices with zero in-degree have no dependencies—safe to place first. After "removing" them, new vertices become zero in-degree. Process continues until all vertices placed or cycle detected.

### Kahn's Complexity

- Time: O(V + E)
- Space: O(V)

## DFS-Based Topological Sort

Post-order DFS naturally gives reverse topological order.

>[!example]- C++
>```cpp
>bool dfs(int node, vector<vector<int>>& graph, vector<int>& color, vector<int>& result) {
>    color[node] = 1; // GRAY (Processing)
>    
>    for (int neighbor : graph[node]) {
>        if (color[neighbor] == 1) return true; // Cycle detected
>        if (color[neighbor] == 0) {
>            if (dfs(neighbor, graph, color, result)) return true;
>        }
>    }
>    
>    color[node] = 2; // BLACK (Done)
>    result.push_back(node);
>    return false;
>}
>
>vector<int> topologicalSortDFS(int n, vector<pair<int, int>>& edges) {
>    vector<vector<int>> graph(n);
>    for (const auto& [u, v] : edges) graph[u].push_back(v);
>    
>    vector<int> color(n, 0); // 0: WHITE, 1: GRAY, 2: BLACK
>    vector<int> result;
>    
>    for (int i = 0; i < n; i++) {
>        if (color[i] == 0) {
>            if (dfs(i, graph, color, result)) return {}; // Cycle
>        }
>    }
>    
>    reverse(result.begin(), result.end());
>    return result;
>}
>```

>[!example]- Java
>```java
>private boolean dfs(int node, List<List<Integer>> graph, int[] color, List<Integer> result) {
>    color[node] = 1; // GRAY
>    
>    for (int neighbor : graph.get(node)) {
>        if (color[neighbor] == 1) return true; // Cycle
>        if (color[neighbor] == 0) {
>            if (dfs(neighbor, graph, color, result)) return true;
>        }
>    }
>    
>    color[node] = 2; // BLACK
>    result.add(node);
>    return false;
>}
>
>public int[] topologicalSortDFS(int n, int[][] edges) {
>    List<List<Integer>> graph = new ArrayList<>();
>    for (int i = 0; i < n; i++) graph.add(new ArrayList<>());
>    for (int[] edge : edges) graph.get(edge[0]).add(edge[1]);
>    
>    int[] color = new int[n];
>    List<Integer> resultList = new ArrayList<>();
>    
>    for (int i = 0; i < n; i++) {
>        if (color[i] == 0) {
>            if (dfs(i, graph, color, resultList)) return new int[0];
>        }
>    }
>    
>    Collections.reverse(resultList);
>    return resultList.stream().mapToInt(i -> i).toArray();
>}
>```

>[!example]- Python
>```python
>def topological_sort_dfs(n, edges):
>    """
>    Returns: topological ordering, or empty list if cycle exists
>    """
>    graph = [[] for _ in range(n)]
>    for u, v in edges:
>        graph[u].append(v)
>
>    WHITE, GRAY, BLACK = 0, 1, 2
>    color = [WHITE] * n
>    result = []
>    has_cycle = [False]
>
>    def dfs(node):
>        if has_cycle[0]:
>            return
>
>        color[node] = GRAY  # Processing
>
>        for neighbor in graph[node]:
>            if color[neighbor] == GRAY:  # Back edge = cycle
>                has_cycle[0] = True
>                return
>            if color[neighbor] == WHITE:
>                dfs(neighbor)
>
>        color[node] = BLACK  # Done
>        result.append(node)  # Post-order
>
>    for i in range(n):
>        if color[i] == WHITE:
>            dfs(i)
>
>    if has_cycle[0]:
>        return []
>
>    return result[::-1]  # Reverse post-order
>```

>[!example]- JavaScript
>```javascript
>function topologicalSortDFS(n, edges) {
>    const graph = Array.from({ length: n }, () => []);
>    for (const [u, v] of edges) graph[u].push(v);
>    
>    const WHITE = 0, GRAY = 1, BLACK = 2;
>    const color = new Array(n).fill(WHITE);
>    const result = [];
>    let hasCycle = false;
>    
>    function dfs(node) {
>        if (hasCycle) return;
>        color[node] = GRAY;
>        
>        for (const neighbor of graph[node]) {
>            if (color[neighbor] === GRAY) {
>                hasCycle = true;
>                return;
>            }
>            if (color[neighbor] === WHITE) {
>                dfs(neighbor);
>            }
>        }
>        
>        color[node] = BLACK;
>        result.push(node);
>    }
>    
>    for (let i = 0; i < n; i++) {
>        if (color[i] === WHITE) {
>            dfs(i);
>            if (hasCycle) return [];
>        }
>    }
>    
>    return result.reverse();
>}
>```

### Why It Works

In DFS, a node is added to result after all its descendants are processed. Reversing gives an order where each node comes before its descendants.

## Comparison

| Aspect | Kahn's (BFS) | DFS-based |
|--------|--------------|-----------|
| Cycle detection | Count processed vertices | Track GRAY nodes |
| Output order | Natural order | Needs reversal |
| Level information | Available (process by levels) | Not directly |
| Lexicographically smallest | Use min-heap instead of queue | Complex |

## Applications

### Course Schedule

```python
def can_finish(num_courses, prerequisites):
    """Return True if all courses can be completed."""
    return len(topological_sort_kahn(num_courses, prerequisites)) == num_courses
```

### Course Schedule II

```python
def find_order(num_courses, prerequisites):
    """Return a valid course order, or empty list if impossible."""
    return topological_sort_kahn(num_courses, prerequisites)
```

### Alien Dictionary

```python
def alien_order(words):
    """Derive character ordering from sorted word list."""
    # Build graph from adjacent word comparisons
    graph = {c: set() for word in words for c in word}
    in_degree = {c: 0 for c in graph}

    for i in range(len(words) - 1):
        w1, w2 = words[i], words[i + 1]
        # Check for invalid input: prefix comes after full word
        if len(w1) > len(w2) and w1[:len(w2)] == w2:
            return ""

        for c1, c2 in zip(w1, w2):
            if c1 != c2:
                if c2 not in graph[c1]:
                    graph[c1].add(c2)
                    in_degree[c2] += 1
                break

    # Kahn's algorithm
    queue = deque([c for c in in_degree if in_degree[c] == 0])
    result = []

    while queue:
        c = queue.popleft()
        result.append(c)
        for neighbor in graph[c]:
            in_degree[neighbor] -= 1
            if in_degree[neighbor] == 0:
                queue.append(neighbor)

    if len(result) != len(graph):
        return ""  # Cycle

    return ''.join(result)
```

### Parallel Job Scheduling

Topological sort gives a valid sequential order. For parallel execution, process all zero in-degree vertices simultaneously (level by level).

```python
def parallel_schedule(n, edges):
    """Return tasks grouped by parallel execution rounds."""
    graph = [[] for _ in range(n)]
    in_degree = [0] * n
    for u, v in edges:
        graph[u].append(v)
        in_degree[v] += 1

    queue = deque([i for i in range(n) if in_degree[i] == 0])
    rounds = []

    while queue:
        round_tasks = []
        for _ in range(len(queue)):  # Process current level
            node = queue.popleft()
            round_tasks.append(node)
            for neighbor in graph[node]:
                in_degree[neighbor] -= 1
                if in_degree[neighbor] == 0:
                    queue.append(neighbor)
        rounds.append(round_tasks)

    return rounds
```

## Common Pitfalls

1. **Not detecting cycles**: Always verify all vertices are processed
2. **Forgetting DAG requirement**: Topological sort only exists for DAGs
3. **Expecting unique order**: Multiple valid orderings may exist
4. **Wrong edge direction**: Clarify whether edge means "before" or "after"

## Key Interview Problems

| Problem | Variant | Difficulty | LeetCode Link |
| --------- | --------- | ------------ | --- |
| Course Schedule | Cycle detection | Medium | [Link](https://leetcode.com/problems/course-schedule/) |
| Course Schedule II | Return ordering | Medium | [Link](https://leetcode.com/problems/course-schedule-ii/) |
| Alien Dictionary | Build graph from constraints | Hard | [Link](https://leetcode.com/problems/alien-dictionary/) |
| Parallel Courses | Minimum rounds | Medium | [Link](https://leetcode.com/problems/parallel-courses/) |
| Sequence Reconstruction | Unique ordering check | Medium | [Link](https://leetcode.com/problems/sequence-reconstruction/) |
