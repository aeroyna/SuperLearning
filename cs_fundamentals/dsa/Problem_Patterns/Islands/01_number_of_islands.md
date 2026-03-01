# Number of Islands (Connected Components)

The "Number of Islands" problem is the most classic example of matrix traversal. It asks us to count the number of connected components in a grid.

## Problem Statement
Given an `m x n` 2D binary grid `grid` which represents a map of '1's (land) and '0's (water), return the number of islands.
An island is surrounded by water and is formed by connecting adjacent lands horizontally or vertically.

## Approach: DFS / BFS

We iterate through every cell in the grid.
1.  If we encounter a `'1'` (land) that hasn't been visited yet, it means we found a new island. Increment the counter.
2.  Start a traversal (DFS or BFS) from that cell to find and mark all connected land cells as visited (e.g., change '1' to '0' or '#').
3.  Continue the iteration.

## Implementation (DFS)

>[!example]- C++
>```cpp
>void dfs(vector<vector<char>>& grid, int r, int c) {
>    int rows = grid.size();
>    int cols = grid[0].size();
>    
>    // Boundary check and visited check
>    if (r < 0 || c < 0 || r >= rows || c >= cols || grid[r][c] == '0') {
>        return;
>    }
>    
>    grid[r][c] = '0'; // Mark as visited (sink the island)
>    
>    // Visit 4 neighbors
>    dfs(grid, r + 1, c);
>    dfs(grid, r - 1, c);
>    dfs(grid, r, c + 1);
>    dfs(grid, r, c - 1);
>}
>
>int numIslands(vector<vector<char>>& grid) {
>    if (grid.empty()) return 0;
>    int count = 0;
>    for (int i = 0; i < grid.size(); i++) {
>        for (int j = 0; j < grid[0].size(); j++) {
>            if (grid[i][j] == '1') {
>                count++;
>                dfs(grid, i, j);
>            }
>        }
>    }
>    return count;
>}
>```

>[!example]- Java
>```java
>class Solution {
>    public int numIslands(char[][] grid) {
>        if (grid == null || grid.length == 0) return 0;
>        int count = 0;
>        for (int i = 0; i < grid.length; i++) {
>            for (int j = 0; j < grid[0].length; j++) {
>                if (grid[i][j] == '1') {
>                    count++;
>                    dfs(grid, i, j);
>                }
>            }
>        }
>        return count;
>    }
>    
>    private void dfs(char[][] grid, int r, int c) {
>        if (r < 0 || c < 0 || r >= grid.length || c >= grid[0].length || grid[r][c] == '0') {
>            return;
>        }
>        
>        grid[r][c] = '0'; // Mark visited
>        
>        dfs(grid, r + 1, c);
>        dfs(grid, r - 1, c);
>        dfs(grid, r, c + 1);
>        dfs(grid, r, c - 1);
>    }
>}
>```

>[!example]- Python
>```python
>def numIslands(grid):
>    if not grid:
>        return 0
>        
>    rows, cols = len(grid), len(grid[0])
>    count = 0
>    
>    def dfs(r, c):
>        if r < 0 or c < 0 or r >= rows or c >= cols or grid[r][c] == '0':
>            return
>        
>        grid[r][c] = '0' # Mark visited
>        
>        dfs(r + 1, c)
>        dfs(r - 1, c)
>        dfs(r, c + 1)
>        dfs(r, c - 1)
>            
>    for r in range(rows):
>        for c in range(cols):
>            if grid[r][c] == '1':
>                count += 1
>                dfs(r, c)
>                
>    return count
>```

>[!example]- JavaScript
>```javascript
>var numIslands = function(grid) {
>    if (!grid || grid.length === 0) return 0;
>    
>    const rows = grid.length;
>    const cols = grid[0].length;
>    let count = 0;
>    
>    const dfs = (r, c) => {
>        if (r < 0 || c < 0 || r >= rows || c >= cols || grid[r][c] === '0') {
>            return;
>        }
>        
>        grid[r][c] = '0'; // Mark visited
>        
>        dfs(r + 1, c);
>        dfs(r - 1, c);
>        dfs(r, c + 1);
>        dfs(r, c - 1);
>    };
>    
>    for (let r = 0; r < rows; r++) {
>        for (let c = 0; c < cols; c++) {
>            if (grid[r][c] === '1') {
>                count++;
>                dfs(r, c);
>            }
>        }
>    }
>    return count;
>};
>```

## Complexity
- **Time**: `O(M * N)`. We visit each cell at most once.
- **Space**: `O(M * N)` worst case stack space (if the entire grid is one island).
