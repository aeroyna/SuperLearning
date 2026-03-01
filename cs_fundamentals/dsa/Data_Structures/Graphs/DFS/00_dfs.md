# Depth-First Search (DFS)

DFS explores as far as possible along each branch before backtracking. It's implemented naturally with recursion (using the call stack) or explicitly with a stack data structure.

## Overview

DFS characteristics:
- Uses a stack (explicit or call stack)
- Goes deep before going wide
- Memory efficient for deep graphs
- Time: O(V + E), Space: O(V) for visited, O(h) for stack where h = max depth

## Topics

- [11.3.1 DFS on Graphs](01_dfs_on_graphs.md)

## Core Templates

### Recursive DFS

>[!example]- C++
>```cpp
>void dfs(unordered_map<int, vector<int>>& graph, int node, unordered_set<int>& visited) {
>    visited.insert(node);
>    process(node);
>    
>    for (int neighbor : graph[node]) {
>        if (visited.find(neighbor) == visited.end()) {
>            dfs(graph, neighbor, visited);
>        }
>    }
>}
>```

>[!example]- Java
>```java
>void dfs(Map<Integer, List<Integer>> graph, int node, Set<Integer> visited) {
>    visited.add(node);
>    process(node);
>    
>    for (int neighbor : graph.getOrDefault(node, new ArrayList<>())) {
>        if (!visited.contains(neighbor)) {
>            dfs(graph, neighbor, visited);
>        }
>    }
>}
>```

>[!example]- Python
>```python
>def dfs_recursive(graph, start):
>    visited = set()
>
>    def dfs(node):
>        visited.add(node)
>        process(node)
>
>        for neighbor in graph[node]:
>            if neighbor not in visited:
>                dfs(neighbor)
>
>    dfs(start)
>    return visited
>```

>[!example]- JavaScript
>```javascript
>function dfsRecursive(graph, start) {
>    const visited = new Set();
>
>    function dfs(node) {
>        visited.add(node);
>        process(node);
>
>        const neighbors = graph[node] || [];
>        for (const neighbor of neighbors) {
>            if (!visited.has(neighbor)) {
>                dfs(neighbor);
>            }
>        }
>    }
>
>    dfs(start);
>    return visited;
>}
>```

### Iterative DFS

>[!example]- C++
>```cpp
>void dfsIterative(unordered_map<int, vector<int>>& graph, int start) {
>    unordered_set<int> visited;
>    stack<int> s;
>    s.push(start);
>    
>    while (!s.empty()) {
>        int node = s.top();
>        s.pop();
>        
>        if (visited.count(node)) continue;
>        visited.insert(node);
>        process(node);
>        
>        for (int neighbor : graph[node]) {
>            if (!visited.count(neighbor)) {
>                s.push(neighbor);
>            }
>        }
>    }
>}
>```

>[!example]- Java
>```java
>void dfsIterative(Map<Integer, List<Integer>> graph, int start) {
>    Set<Integer> visited = new HashSet<>();
>    Stack<Integer> stack = new Stack<>();
>    stack.push(start);
>    
>    while (!stack.isEmpty()) {
>        int node = stack.pop();
>        
>        if (visited.contains(node)) continue;
>        visited.add(node);
>        process(node);
>        
>        for (int neighbor : graph.getOrDefault(node, new ArrayList<>())) {
>            if (!visited.contains(neighbor)) {
>                stack.push(neighbor);
>            }
>        }
>    }
>}
>```

>[!example]- Python
>```python
>def dfs_iterative(graph, start):
>    visited = set()
>    stack = [start]
>
>    while stack:
>        node = stack.pop()
>        if node in visited:
>            continue
>        visited.add(node)
>        process(node)
>
>        for neighbor in graph[node]:
>            if neighbor not in visited:
>                stack.append(neighbor)
>
>    return visited
>```

>[!example]- JavaScript
>```javascript
>function dfsIterative(graph, start) {
>    const visited = new Set();
>    const stack = [start];
>    
>    while (stack.length > 0) {
>        const node = stack.pop();
>        
>        if (visited.has(node)) continue;
>        visited.add(node);
>        process(node);
>        
>        const neighbors = graph[node] || [];
>        for (const neighbor of neighbors) {
>            if (!visited.has(neighbor)) {
>                stack.push(neighbor);
>            }
>        }
>    }
>    return visited;
>}
>```

**Note**: Iterative version may process neighbors in different order than recursive due to stack LIFO behavior.

## DFS Execution Trace

```
Graph:
    A --- B
    |     |
    C --- D --- E

DFS from A (going alphabetically):
Visit A, recurse to B
Visit B, recurse to D
Visit D, recurse to C (already visited via A? No, visit)
Visit C (neighbors A, D already visited), backtrack
Back to D, recurse to E
Visit E, backtrack all the way

Order: A → B → D → C → E (depth-first)
```

## DFS States for Cycle Detection

Three states track progress through nodes:

```python
WHITE = 0  # Unvisited
GRAY = 1   # Being processed (in current path)
BLACK = 2  # Fully processed

def has_cycle_directed(graph):
    color = {node: WHITE for node in graph}

    def dfs(node):
        color[node] = GRAY  # Start processing

        for neighbor in graph[node]:
            if color[neighbor] == GRAY:
                return True  # Back edge = cycle
            if color[neighbor] == WHITE:
                if dfs(neighbor):
                    return True

        color[node] = BLACK  # Done processing
        return False

    for node in graph:
        if color[node] == WHITE:
            if dfs(node):
                return True
    return False
```

**Why three colors**: In directed graphs, reaching a BLACK node isn't a cycle (just a cross edge). Only reaching a GRAY node (in current DFS path) indicates a cycle.

## Finding All Paths

```python
def all_paths(graph, start, end):
    paths = []

    def dfs(node, path):
        if node == end:
            paths.append(path[:])  # Copy current path
            return

        for neighbor in graph[node]:
            if neighbor not in path:  # Avoid cycles
                path.append(neighbor)
                dfs(neighbor, path)
                path.pop()  # Backtrack

    dfs(start, [start])
    return paths
```

## DFS for Connected Components

```python
def count_components(graph, n):
    visited = set()
    count = 0

    def dfs(node):
        visited.add(node)
        for neighbor in graph[node]:
            if neighbor not in visited:
                dfs(neighbor)

    for node in range(n):
        if node not in visited:
            dfs(node)
            count += 1

    return count
```

## DFS on Trees vs Graphs

### Trees (No Cycle Check Needed)

```python
def dfs_tree(root):
    if not root:
        return

    process(root)
    dfs_tree(root.left)
    dfs_tree(root.right)
```

### Graphs (Must Track Visited)

```python
def dfs_graph(graph, start):
    visited = {start}

    def dfs(node):
        process(node)
        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                dfs(neighbor)

    dfs(start)
```

## DFS with Entry/Exit Times

Track when nodes are entered and exited—useful for various algorithms:

```python
def dfs_with_times(graph, start):
    time = [0]
    entry = {}
    exit = {}

    def dfs(node):
        time[0] += 1
        entry[node] = time[0]

        for neighbor in graph[node]:
            if neighbor not in entry:
                dfs(neighbor)

        time[0] += 1
        exit[node] = time[0]

    dfs(start)
    return entry, exit
```

**Applications**:
- Check if u is ancestor of v: `entry[u] < entry[v] < exit[v] < exit[u]`
- Topological sort: Sort by decreasing exit time

## Grid DFS

```python
def dfs_grid(grid, r, c, visited):
    rows, cols = len(grid), len(grid[0])

    if (r < 0 or r >= rows or c < 0 or c >= cols or
        (r, c) in visited or grid[r][c] == '#'):
        return

    visited.add((r, c))

    dfs_grid(grid, r + 1, c, visited)
    dfs_grid(grid, r - 1, c, visited)
    dfs_grid(grid, r, c + 1, visited)
    dfs_grid(grid, r, c - 1, visited)
```

## BFS vs DFS Decision

| Aspect | BFS | DFS |
|--------|-----|-----|
| Data structure | Queue | Stack/recursion |
| Order | Level by level | Branch by branch |
| Shortest path (unweighted) | Yes | No |
| Memory | O(width) | O(depth) |
| Finding any path | Either | Often simpler |
| Topological sort | Yes (Kahn's) | Yes (reverse postorder) |
| Cycle detection | Less natural | Natural with colors |

**Rule of thumb**:
- Need shortest path → BFS
- Need to explore all possibilities → DFS
- Tree structure → Usually DFS
- Level-by-level processing → BFS

## Common Pitfalls

1. **Stack overflow**: Deep recursion exceeds limit—use iterative for very deep graphs
2. **Forgetting visited in graphs**: Infinite loop
3. **Modifying data during traversal**: Can cause unexpected behavior
4. **Not handling disconnected graphs**: Start DFS from each component

## Key Interview Problems

| Problem | Pattern | Difficulty | LeetCode Link |
| --------- | --------- | ------------ | --- |
| Number of Islands | Grid DFS | Medium | [Link](https://leetcode.com/problems/number-of-islands/) |
| Clone Graph | DFS + hash map | Medium | [Link](https://leetcode.com/problems/clone-graph/) |
| Course Schedule | Cycle detection | Medium | [Link](https://leetcode.com/problems/course-schedule/) |
| All Paths From Source to Target | Path enumeration | Medium | [Link](https://leetcode.com/problems/all-paths-from-source-to-target/) |
| Surrounded Regions | Border-connected DFS | Medium | [Link](https://leetcode.com/problems/surrounded-regions/) |
| Pacific Atlantic Water Flow | Multi-source DFS | Medium | [Link](https://leetcode.com/problems/pacific-atlantic-water-flow/) |
