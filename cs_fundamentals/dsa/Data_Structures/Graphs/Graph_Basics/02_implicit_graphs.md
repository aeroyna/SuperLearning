## Implicit Graphs (Grids)

In many interview problems, a graph is not given to you explicitly as a set of nodes and edges. Instead, the graph is **implicit** in the problem's structure. The most common form of an implicit graph is a **2D grid or matrix**.

### Core Idea
When you see a problem involving a grid where you can move from one cell to another based on certain rules, you should immediately think of it as a graph problem.

- **Nodes**: Each cell `(row, col)` in the grid is a node.
- **Edges**: Edges connect a cell to its neighbors. The definition of "neighbor" depends on the problem. It could be:
    - **4-directional**: Up, Down, Left, Right.
    - **8-directional**: The 4 cardinal directions plus the 4 diagonals.
- **Traversal**: You don't need to build an explicit adjacency list for the entire grid beforehand. Instead, you can find the neighbors of any given cell `(r, c)` **on the fly** during your traversal (DFS or BFS).

### On-the-Fly Neighbor Calculation
This is the key technique for implicit graphs. Instead of a pre-built `graph[node]` lookup, you have a function or a loop that calculates neighbors.

>[!example]- C++
>```cpp
>#include <vector>
>using namespace std;
>
>vector<pair<int, int>> getNeighbors(int row, int col, int height, int width) {
>    vector<pair<int, int>> neighbors;
>    // Define the possible moves (e.g., 4-directional)
>    vector<pair<int, int>> directions = {{0, 1}, {0, -1}, {1, 0}, {-1, 0}}; // Right, Left, Down, Up
>
>    for (const auto& [dr, dc] : directions) {
>        int newRow = row + dr;
>        int newCol = col + dc;
>
>        // Check if the new coordinates are within the grid boundaries
>        if (newRow >= 0 && newRow < height && newCol >= 0 && newCol < width) {
>            neighbors.push_back({newRow, newCol});
>        }
>    }
>
>    return neighbors;
>}
>```

>[!example]- Java
>```java
>import java.util.*;
>
>public List<int[]> getNeighbors(int row, int col, int height, int width) {
>    List<int[]> neighbors = new ArrayList<>();
>    // Define the possible moves (e.g., 4-directional)
>    int[][] directions = {{0, 1}, {0, -1}, {1, 0}, {-1, 0}}; // Right, Left, Down, Up
>
>    for (int[] dir : directions) {
>        int newRow = row + dir[0];
>        int newCol = col + dir[1];
>
>        // Check if the new coordinates are within the grid boundaries
>        if (newRow >= 0 && newRow < height && newCol >= 0 && newCol < width) {
>            neighbors.add(new int[]{newRow, newCol});
>        }
>    }
>
>    return neighbors;
>}
>```

>[!example]- Python
>```python
>def get_neighbors(row, col, height, width):
>    neighbors = []
>    # Define the possible moves (e.g., 4-directional)
>    directions = [(0, 1), (0, -1), (1, 0), (-1, 0)] # Right, Left, Down, Up
>
>    for dr, dc in directions:
>        new_row, new_col = row + dr, col + dc
>
>        # Check if the new coordinates are within the grid boundaries
>        if 0 <= new_row < height and 0 <= new_col < width:
>            neighbors.append((new_row, new_col))
>
>    return neighbors
>```

>[!example]- JavaScript
>```javascript
>function getNeighbors(row, col, height, width) {
>    const neighbors = [];
>    // Define the possible moves (e.g., 4-directional)
>    const directions = [[0, 1], [0, -1], [1, 0], [-1, 0]]; // Right, Left, Down, Up
>
>    for (const [dr, dc] of directions) {
>        const newRow = row + dr;
>        const newCol = col + dc;
>
>        // Check if the new coordinates are within the grid boundaries
>        if (newRow >= 0 && newRow < height && newCol >= 0 && newCol < width) {
>            neighbors.push([newRow, newCol]);
>        }
>    }
>
>    return neighbors;
>}
>```

You would call this helper function inside your main DFS or BFS loop whenever you need to explore from the current cell.

### Classic Examples
Recognizing the implicit graph structure is the key first step to solving these famous problems.

- **Number of Islands (LeetCode #200)**: The grid is a graph where land cells ('1') are nodes, and adjacent land cells are connected. The goal is to find the number of connected components in this graph.
- **Shortest Path in Binary Matrix (LeetCode #1091)**: The grid is a graph where empty cells ('0') are nodes. The goal is to find the shortest path from the top-left to the bottom-right, a classic BFS application.
- **Word Search (LeetCode #79)**: The grid of letters is a graph. The problem asks you to find a path in the graph that spells out a given word. This is a classic application of backtracking (a form of DFS).

Treating a grid as a graph allows you to apply all the standard graph traversal algorithms (DFS, BFS) and patterns to solve grid-based problems.
