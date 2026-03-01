# Matrix BFS (Shortest Path & Rotting Oranges)

Breadth-First Search (BFS) on a matrix is the standard algorithm for finding the **shortest path** in an unweighted grid or for simulating simultaneous expansion (like fire spreading or rotting oranges).

## Problem: Rotting Oranges
Given a grid where `0` is empty, `1` is a fresh orange, and `2` is a rotten orange. Every minute, any fresh orange adjacent (4-directionally) to a rotten orange becomes rotten. Return the minimum number of minutes until no fresh orange remains.

## Approach: Multi-Source BFS

Instead of starting BFS from a single node, we start from **all** rotten oranges simultaneously.
1.  Initialize a `Queue` and add all coordinates of rotten oranges.
2.  Count `fresh` oranges.
3.  Process the queue level by level (minutes).
    - For each cell in the current level, infect its neighbors.
    - If a neighbor becomes rotten, add it to the queue and decrement `fresh` count.
4.  Return minutes if `fresh == 0`, otherwise `-1`.

## Implementation

>[!example]- C++
>```cpp
>int orangesRotting(vector<vector<int>>& grid) {
>    int rows = grid.size();
>    int cols = grid[0].size();
>    queue<pair<int, int>> q;
>    int fresh = 0;
>    
>    for(int i=0; i<rows; i++) {
>        for(int j=0; j<cols; j++) {
>            if(grid[i][j] == 2) q.push({i, j});
>            else if(grid[i][j] == 1) fresh++;
>        }
>    }
>    
>    if(fresh == 0) return 0;
>    
>    int minutes = 0;
>    int dirs[4][2] = {{0,1}, {0,-1}, {1,0}, {-1,0}};
>    
>    while(!q.empty()) {
>        int size = q.size();
>        bool changed = false;
>        while(size--) {
>            auto [r, c] = q.front(); q.pop();
>            for(auto& d : dirs) {
>                int nr = r + d[0];
>                int nc = c + d[1];
>                if(nr >= 0 && nc >= 0 && nr < rows && nc < cols && grid[nr][nc] == 1) {
>                    grid[nr][nc] = 2;
>                    q.push({nr, nc});
>                    fresh--;
>                    changed = true;
>                }
>            }
>        }
>        if(changed) minutes++;
>    }
>    
>    return fresh == 0 ? minutes : -1;
>}
>```

>[!example]- Java
>```java
>public int orangesRotting(int[][] grid) {
>    int rows = grid.length;
>    int cols = grid[0].length;
>    Queue<int[]> q = new LinkedList<>();
>    int fresh = 0;
>    
>    for(int i=0; i<rows; i++) {
>        for(int j=0; j<cols; j++) {
>            if(grid[i][j] == 2) q.offer(new int[]{i, j});
>            else if(grid[i][j] == 1) fresh++;
>        }
>    }
>    
>    if(fresh == 0) return 0;
>    
>    int minutes = 0;
>    int[][] dirs = {{0,1}, {0,-1}, {1,0}, {-1,0}};
>    
>    while(!q.isEmpty()) {
>        int size = q.size();
>        boolean changed = false;
>        for(int i=0; i<size; i++) {
>            int[] cell = q.poll();
>            for(int[] d : dirs) {
>                int nr = cell[0] + d[0];
>                int nc = cell[1] + d[1];
>                if(nr >= 0 && nc >= 0 && nr < rows && nc < cols && grid[nr][nc] == 1) {
>                    grid[nr][nc] = 2;
>                    q.offer(new int[]{nr, nc});
>                    fresh--;
>                    changed = true;
>                }
>            }
>        }
>        if(changed) minutes++;
>    }
>    
>    return fresh == 0 ? minutes : -1;
>}
>```

>[!example]- Python
>```python
>from collections import deque
>
>def orangesRotting(grid):
>    rows, cols = len(grid), len(grid[0])
>    q = deque()
>    fresh = 0
>    
>    for r in range(rows):
>        for c in range(cols):
>            if grid[r][c] == 2:
>                q.append((r, c))
>            elif grid[r][c] == 1:
>                fresh += 1
>                
>    if fresh == 0: return 0
>    
>    minutes = 0
>    dirs = [(0,1), (0,-1), (1,0), (-1,0)]
>    
>    while q and fresh > 0:
>        minutes += 1
>        for _ in range(len(q)):
>            r, c = q.popleft()
>            for dr, dc in dirs:
>                nr, nc = r + dr, c + dc
>                if 0 <= nr < rows and 0 <= nc < cols and grid[nr][nc] == 1:
>                    grid[nr][nc] = 2
>                    q.append((nr, nc))
>                    fresh -= 1
>                    
>    return minutes if fresh == 0 else -1
>```

>[!example]- JavaScript
>```javascript
>var orangesRotting = function(grid) {
>    const rows = grid.length;
>    const cols = grid[0].length;
>    const q = [];
>    let fresh = 0;
>    
>    for(let r=0; r<rows; r++) {
>        for(let c=0; c<cols; c++) {
>            if(grid[r][c] === 2) q.push([r, c]);
>            else if(grid[r][c] === 1) fresh++;
>        }
>    }
>    
>    if(fresh === 0) return 0;
>    
>    let minutes = 0;
>    const dirs = [[0,1], [0,-1], [1,0], [-1,0]];
>    let head = 0; // Simulate queue with array pointer
>    
>    while(head < q.length) {
>        const size = q.length - head;
>        let changed = false;
>        
>        for(let i=0; i<size; i++) {
>            const [r, c] = q[head++];
>            for(const [dr, dc] of dirs) {
>                const nr = r + dr;
>                const nc = c + dc;
>                if(nr >= 0 && nc >= 0 && nr < rows && nc < cols && grid[nr][nc] === 1) {
>                    grid[nr][nc] = 2;
>                    q.push([nr, nc]);
>                    fresh--;
>                    changed = true;
>                }
>            }
>        }
>        if(changed) minutes++;
>    }
>    
>    return fresh === 0 ? minutes : -1;
>};
>```

## Key Insight
Using a standard queue size loop (`for _ in range(len(q))`) allows us to process the graph layer-by-layer, which perfectly tracks the "time" or "distance" from the source(s).
