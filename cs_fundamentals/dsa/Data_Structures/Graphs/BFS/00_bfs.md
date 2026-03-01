# Breadth-First Search (BFS)

BFS explores a graph level by level, visiting all vertices at distance k before any at distance k+1. This property makes BFS optimal for finding shortest paths in unweighted graphs.

## Overview

BFS characteristics:
- Uses a queue (FIFO order)
- Visits nodes in order of distance from source
- Guaranteed shortest path in unweighted graphs
- Time: O(V + E), Space: O(V)

## Topics

- [11.2.1 BFS on Graphs](01_bfs_on_graphs.md)

## Core Template

>[!example]- C++
>```cpp
>void bfs(unordered_map<int, vector<int>>& graph, int start) {
>    unordered_set<int> visited;
>    queue<int> q;
>    
>    visited.insert(start);
>    q.push(start);
>    
>    while (!q.empty()) {
>        int node = q.front();
>        q.pop();
>        process(node);
>        
>        for (int neighbor : graph[node]) {
>            if (visited.find(neighbor) == visited.end()) {
>                visited.insert(neighbor);
>                q.push(neighbor);
>            }
>        }
>    }
>}
>```

>[!example]- Java
>```java
>void bfs(Map<Integer, List<Integer>> graph, int start) {
>    Set<Integer> visited = new HashSet<>();
>    Queue<Integer> queue = new LinkedList<>();
>    
>    visited.add(start);
>    queue.offer(start);
>    
>    while (!queue.isEmpty()) {
>        int node = queue.poll();
>        process(node);
>        
>        for (int neighbor : graph.getOrDefault(node, new ArrayList<>())) {
>            if (!visited.contains(neighbor)) {
>                visited.add(neighbor);
>                queue.offer(neighbor);
>            }
>        }
>    }
>}
>```

>[!example]- Python
>```python
>from collections import deque
>
>def bfs(graph, start):
>    visited = {start}
>    queue = deque([start])
>
>    while queue:
>        node = queue.popleft()
>        process(node)
>
>        for neighbor in graph[node]:
>            if neighbor not in visited:
>                visited.add(neighbor)
>                queue.append(neighbor)
>```

>[!example]- JavaScript
>```javascript
>function bfs(graph, start) {
>    const visited = new Set([start]);
>    const queue = [start];
>    
>    while (queue.length > 0) {
>        const node = queue.shift();
>        process(node);
>        
>        // Assuming graph is Map or Object
>        const neighbors = graph[node] || [];
>        for (const neighbor of neighbors) {
>            if (!visited.has(neighbor)) {
>                visited.add(neighbor);
>                queue.push(neighbor);
>            }
>        }
>    }
>}
>```

**Key insight**: Mark visited when enqueueing, not when dequeuing. This prevents the same node from being added multiple times.

## BFS Execution Trace

```
Graph:
    A --- B
    |     |
    C --- D --- E

BFS from A:
Queue: [A]        Visited: {A}
Dequeue A, enqueue B, C
Queue: [B, C]     Visited: {A, B, C}
Dequeue B, enqueue D (C already visited)
Queue: [C, D]     Visited: {A, B, C, D}
Dequeue C (D already visited)
Queue: [D]
Dequeue D, enqueue E
Queue: [E]        Visited: {A, B, C, D, E}
Dequeue E, done

Order: A → B → C → D → E (level by level)
```

## Shortest Path (Unweighted)

```python
def shortest_path(graph, start, end):
    if start == end:
        return [start]

    visited = {start}
    queue = deque([(start, [start])])  # (node, path)

    while queue:
        node, path = queue.popleft()

        for neighbor in graph[node]:
            if neighbor == end:
                return path + [neighbor]
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append((neighbor, path + [neighbor]))

    return []  # No path exists
```

**Optimized (parent tracking)**:

```python
def shortest_path_optimized(graph, start, end):
    if start == end:
        return [start]

    visited = {start}
    parent = {start: None}
    queue = deque([start])

    while queue:
        node = queue.popleft()

        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                parent[neighbor] = node
                if neighbor == end:
                    # Reconstruct path
                    path = []
                    current = end
                    while current is not None:
                        path.append(current)
                        current = parent[current]
                    return path[::-1]
                queue.append(neighbor)

    return []
```

## Level-Order BFS

Process nodes level by level:

```python
def bfs_levels(graph, start):
    visited = {start}
    queue = deque([start])
    levels = []

    while queue:
        level_size = len(queue)
        current_level = []

        for _ in range(level_size):
            node = queue.popleft()
            current_level.append(node)

            for neighbor in graph[node]:
                if neighbor not in visited:
                    visited.add(neighbor)
                    queue.append(neighbor)

        levels.append(current_level)

    return levels
```

**Distance tracking alternative**:

```python
def bfs_with_distance(graph, start):
    visited = {start}
    queue = deque([(start, 0)])  # (node, distance)

    while queue:
        node, dist = queue.popleft()
        print(f"{node} at distance {dist}")

        for neighbor in graph[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append((neighbor, dist + 1))
```

## Multi-Source BFS

Start from multiple sources simultaneously—useful for "minimum distance to any source":

```python
def multi_source_bfs(grid, sources):
    """Find minimum distance from each cell to nearest source."""
    rows, cols = len(grid), len(grid[0])
    distances = [[float('inf')] * cols for _ in range(rows)]
    queue = deque()

    # Initialize all sources
    for r, c in sources:
        distances[r][c] = 0
        queue.append((r, c))

    while queue:
        r, c = queue.popleft()
        for dr, dc in [(0, 1), (0, -1), (1, 0), (-1, 0)]:
            nr, nc = r + dr, c + dc
            if 0 <= nr < rows and 0 <= nc < cols:
                if distances[nr][nc] > distances[r][c] + 1:
                    distances[nr][nc] = distances[r][c] + 1
                    queue.append((nr, nc))

    return distances
```

**Use cases**: Rotting oranges, walls and gates, 01 matrix.

## Bidirectional BFS

For source-to-target, search from both ends—meets in middle:

```python
def bidirectional_bfs(graph, start, end):
    if start == end:
        return 0

    front_visited = {start}
    back_visited = {end}
    front_queue = deque([start])
    back_queue = deque([end])
    distance = 0

    while front_queue and back_queue:
        distance += 1

        # Expand smaller frontier
        if len(front_queue) <= len(back_queue):
            if expand(graph, front_queue, front_visited, back_visited):
                return distance
        else:
            if expand(graph, back_queue, back_visited, front_visited):
                return distance

    return -1  # No path

def expand(graph, queue, visited, other_visited):
    for _ in range(len(queue)):
        node = queue.popleft()
        for neighbor in graph[node]:
            if neighbor in other_visited:
                return True  # Paths meet
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)
    return False
```

**Complexity improvement**: O(b^(d/2)) vs O(b^d) where b = branching factor, d = distance.

## BFS on Implicit Graphs

### Grid Traversal

```python
def bfs_grid(grid, start):
    rows, cols = len(grid), len(grid[0])
    visited = {start}
    queue = deque([start])

    while queue:
        r, c = queue.popleft()

        for dr, dc in [(0, 1), (0, -1), (1, 0), (-1, 0)]:
            nr, nc = r + dr, c + dc
            if 0 <= nr < rows and 0 <= nc < cols:
                if (nr, nc) not in visited and grid[nr][nc] != '#':
                    visited.add((nr, nc))
                    queue.append((nr, nc))
```

### State Space

```python
def min_moves(start_state, target_state):
    """BFS on state space."""
    visited = {start_state}
    queue = deque([(start_state, 0)])

    while queue:
        state, moves = queue.popleft()

        if state == target_state:
            return moves

        for next_state in get_next_states(state):
            if next_state not in visited:
                visited.add(next_state)
                queue.append((next_state, moves + 1))

    return -1
```

## Common Pitfalls

1. **Adding to visited on dequeue**: Causes duplicate processing
2. **Forgetting visited set**: Infinite loop on cyclic graphs
3. **Using list instead of deque**: `list.pop(0)` is O(n)
4. **Not handling disconnected components**: May need to start BFS from each unvisited node

## Key Interview Problems

| Problem | Pattern | Difficulty | LeetCode Link |
| --------- | --------- | ------------ | --- |
| Binary Tree Level Order | Basic BFS | Medium | [Link](https://leetcode.com/problems/binary-tree-level-order/) |
| Number of Islands | Grid BFS | Medium | [Link](https://leetcode.com/problems/number-of-islands/) |
| Shortest Path in Binary Matrix | Grid shortest path | Medium | [Link](https://leetcode.com/problems/shortest-path-in-binary-matrix/) |
| Word Ladder | State space BFS | Hard | [Link](https://leetcode.com/problems/word-ladder/) |
| Rotting Oranges | Multi-source BFS | Medium | [Link](https://leetcode.com/problems/rotting-oranges/) |
| Open the Lock | State space + pruning | Medium | [Link](https://leetcode.com/problems/open-the-lock/) |
