## Backtracking: Constraint Satisfaction Problems

Another major class of backtracking problems involves **constraint satisfaction**. Unlike generation problems where you find all possible combinations, here the goal is typically to find a **single valid solution** (or sometimes any valid solution) that adheres to a specific set of rules or constraints.

Examples of such problems include solving a Sudoku puzzle, the N-Queens problem, or finding a path through a maze.

### Core Idea: Pruning the Search Space
The "choose, explore, unchoose" template is still central, but the emphasis shifts to the **pruning** step. At each stage of the recursion, you make a choice and then immediately check if that choice violates any of the problem's constraints. If it does, you "prune" that entire branch of the search tree by returning immediately, avoiding a massive amount of unnecessary computation.

The general flow becomes:
1.  Choose a candidate for the next open "spot".
2.  Check if this choice is **valid** according to the rules.
3.  If valid:
    a. Place the candidate.
    b. Recurse to solve for the next spot.
    c. If the recursive call successfully finds a solution, you're often done. Propagate the success up the call stack.
    d. If not, **unchoose** (backtrack) and try the next candidate for the current spot.
4.  If not valid, try the next candidate for the current spot.

### Example: The N-Queens Problem
**Problem**: The N-Queens puzzle is the problem of placing `N` chess queens on an `N×N` chessboard so that no two queens threaten each other. This means no two queens can share the same row, column, or diagonal.

**Backtracking Solution**:
- **State**: We can solve this row by row. Our recursive function will try to place a queen in `row r`.
- **Choices**: At `row r`, the possible choices are any of the `N` columns.
- **Constraints**: Before placing a queen at `(r, c)`, we must check if that square is under attack from any previously placed queens (in rows `0` to `r-1`). A square is under attack if another queen is in the same column, or on the same positive or negative diagonal.

>[!example]- C++
>```cpp
>vector<vector<string>> solve_n_queens(int n) {
>    // The board will store the column position of the queen for each row.
>    // e.g., board[0] = 1 means a queen is at (0, 1).
>    vector<int> board(n, -1);
>    vector<vector<string>> solutions;
>
>    function<bool(int, int)> is_valid = [&](int row, int col) {
>        // Check all previous rows
>        for (int r = 0; r < row; r++) {
>            // Check for same column and same diagonals
>            if (board[r] == col || abs(row - r) == abs(col - board[r])) {
>                return false;
>            }
>        }
>        return true;
>    };
>
>    function<void(int)> backtrack = [&](int row) {
>        // Base Case: If we have successfully placed a queen in every row
>        if (row == n) {
>            // Construct and add the solution to our list
>            vector<string> solution;
>            for (int r = 0; r < n; r++) {
>                solution.push_back(string(board[r], '.') + "Q" + string(n - board[r] - 1, '.'));
>            }
>            solutions.push_back(solution);
>            return;
>        }
>
>        // Iterate through all possible column choices for the current row
>        for (int col = 0; col < n; col++) {
>            // Check if the choice is valid
>            if (is_valid(row, col)) {
>                // Choose
>                board[row] = col;
>                // Explore
>                backtrack(row + 1);
>                // Unchoose (not strictly necessary here as we overwrite board[row],
>                // but good practice to show the backtracking step)
>                board[row] = -1;
>            }
>        }
>    };
>
>    backtrack(0);
>    return solutions;
>}
>```

>[!example]- Java
>```java
>public List<List<String>> solve_n_queens(int n) {
>    // The board will store the column position of the queen for each row.
>    // e.g., board[0] = 1 means a queen is at (0, 1).
>    int[] board = new int[n];
>    Arrays.fill(board, -1);
>    List<List<String>> solutions = new ArrayList<>();
>
>    backtrack(0, n, board, solutions);
>    return solutions;
>}
>
>private boolean is_valid(int row, int col, int[] board) {
>    // Check all previous rows
>    for (int r = 0; r < row; r++) {
>        // Check for same column and same diagonals
>        if (board[r] == col || Math.abs(row - r) == Math.abs(col - board[r])) {
>            return false;
>        }
>    }
>    return true;
>}
>
>private void backtrack(int row, int n, int[] board, List<List<String>> solutions) {
>    // Base Case: If we have successfully placed a queen in every row
>    if (row == n) {
>        // Construct and add the solution to our list
>        List<String> solution = new ArrayList<>();
>        for (int r = 0; r < n; r++) {
>            StringBuilder sb = new StringBuilder();
>            for (int c = 0; c < n; c++) {
>                sb.append(c == board[r] ? "Q" : ".");
>            }
>            solution.add(sb.toString());
>        }
>        solutions.add(solution);
>        return;
>    }
>
>    // Iterate through all possible column choices for the current row
>    for (int col = 0; col < n; col++) {
>        // Check if the choice is valid
>        if (is_valid(row, col, board)) {
>            // Choose
>            board[row] = col;
>            // Explore
>            backtrack(row + 1, n, board, solutions);
>            // Unchoose (not strictly necessary here as we overwrite board[row],
>            // but good practice to show the backtracking step)
>            board[row] = -1;
>        }
>    }
>}
>```

>[!example]- Python
>```python
>def solve_n_queens(n):
>    # The board will store the column position of the queen for each row.
>    # e.g., board[0] = 1 means a queen is at (0, 1).
>    board = [-1] * n
>    solutions = []
>
>    def is_valid(row, col):
>        # Check all previous rows
>        for r in range(row):
>            # Check for same column and same diagonals
>            if board[r] == col or abs(row - r) == abs(col - board[r]):
>                return False
>        return True
>
>    def backtrack(row):
>        # Base Case: If we have successfully placed a queen in every row
>        if row == n:
>            # Construct and add the solution to our list
>            solution = []
>            for r in range(n):
>                solution.append("." * board[r] + "Q" + "." * (n - board[r] - 1))
>            solutions.append(solution)
>            return
>
>        # Iterate through all possible column choices for the current row
>        for col in range(n):
>            # Check if the choice is valid
>            if is_valid(row, col):
>                # Choose
>                board[row] = col
>                # Explore
>                backtrack(row + 1)
>                # Unchoose (not strictly necessary here as we overwrite board[row],
>                # but good practice to show the backtracking step)
>                board[row] = -1
>
>    backtrack(0)
>    return solutions
>```

>[!example]- JavaScript
>```javascript
>function solve_n_queens(n) {
>    // The board will store the column position of the queen for each row.
>    // e.g., board[0] = 1 means a queen is at (0, 1).
>    const board = new Array(n).fill(-1);
>    const solutions = [];
>
>    function is_valid(row, col) {
>        // Check all previous rows
>        for (let r = 0; r < row; r++) {
>            // Check for same column and same diagonals
>            if (board[r] === col || Math.abs(row - r) === Math.abs(col - board[r])) {
>                return false;
>            }
>        }
>        return true;
>    }
>
>    function backtrack(row) {
>        // Base Case: If we have successfully placed a queen in every row
>        if (row === n) {
>            // Construct and add the solution to our list
>            const solution = [];
>            for (let r = 0; r < n; r++) {
>                solution.push(".".repeat(board[r]) + "Q" + ".".repeat(n - board[r] - 1));
>            }
>            solutions.push(solution);
>            return;
>        }
>
>        // Iterate through all possible column choices for the current row
>        for (let col = 0; col < n; col++) {
>            // Check if the choice is valid
>            if (is_valid(row, col)) {
>                // Choose
>                board[row] = col;
>                // Explore
>                backtrack(row + 1);
>                // Unchoose (not strictly necessary here as we overwrite board[row],
>                // but good practice to show the backtracking step)
>                board[row] = -1;
>            }
>        }
>    }
>
>    backtrack(0);
>    return solutions;
>}
>```

This approach systematically explores valid placements. When a placement leads to a dead end (a row where no queen can be placed), the function returns, and the loop continues to the next column choice, effectively "backtracking" from the invalid state.
