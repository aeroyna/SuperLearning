# Backtracking

Backtracking is a technique for finding all (or some) solutions by trying candidates and abandoning those that fail to satisfy constraints.

## Backtracking Template

```cpp
void backtrack(/* state, choices, result */) {
    // Base case: found a solution
    if (/* goal reached */) {
        result.push_back(/* current solution */);
        return;
    }

    for (/* each choice */) {
        if (/* choice is valid */) {
            // Make choice
            // state.add(choice);

            // Recurse
            backtrack(/* updated state */);

            // Undo choice (backtrack)
            // state.remove(choice);
        }
    }
}
```

## Classic Problems

### Subsets

```cpp
std::vector<std::vector<int>> subsets(std::vector<int>& nums) {
    std::vector<std::vector<int>> result;
    std::vector<int> current;

    std::function<void(int)> backtrack = [&](int start) {
        result.push_back(current);

        for (int i = start; i < nums.size(); ++i) {
            current.push_back(nums[i]);
            backtrack(i + 1);
            current.pop_back();
        }
    };

    backtrack(0);
    return result;
}
```

### Permutations

```cpp
std::vector<std::vector<int>> permute(std::vector<int>& nums) {
    std::vector<std::vector<int>> result;
    std::vector<int> current;
    std::vector<bool> used(nums.size(), false);

    std::function<void()> backtrack = [&]() {
        if (current.size() == nums.size()) {
            result.push_back(current);
            return;
        }

        for (int i = 0; i < nums.size(); ++i) {
            if (!used[i]) {
                used[i] = true;
                current.push_back(nums[i]);
                backtrack();
                current.pop_back();
                used[i] = false;
            }
        }
    };

    backtrack();
    return result;
}
```

### Combinations (n choose k)

```cpp
std::vector<std::vector<int>> combine(int n, int k) {
    std::vector<std::vector<int>> result;
    std::vector<int> current;

    std::function<void(int)> backtrack = [&](int start) {
        if (current.size() == k) {
            result.push_back(current);
            return;
        }

        // Pruning: need k - current.size() more elements
        for (int i = start; i <= n - (k - current.size()) + 1; ++i) {
            current.push_back(i);
            backtrack(i + 1);
            current.pop_back();
        }
    };

    backtrack(1);
    return result;
}
```

### N-Queens

```cpp
std::vector<std::vector<std::string>> solveNQueens(int n) {
    std::vector<std::vector<std::string>> result;
    std::vector<std::string> board(n, std::string(n, '.'));
    std::vector<bool> cols(n), diag1(2*n), diag2(2*n);

    std::function<void(int)> backtrack = [&](int row) {
        if (row == n) {
            result.push_back(board);
            return;
        }

        for (int col = 0; col < n; ++col) {
            if (!cols[col] && !diag1[row+col] && !diag2[row-col+n]) {
                board[row][col] = 'Q';
                cols[col] = diag1[row+col] = diag2[row-col+n] = true;

                backtrack(row + 1);

                board[row][col] = '.';
                cols[col] = diag1[row+col] = diag2[row-col+n] = false;
            }
        }
    };

    backtrack(0);
    return result;
}
```

### Word Search in Grid

```cpp
bool exist(std::vector<std::vector<char>>& board, std::string word) {
    int m = board.size(), n = board[0].size();
    std::vector<std::pair<int,int>> dirs = {{0,1},{1,0},{0,-1},{-1,0}};

    std::function<bool(int, int, int)> backtrack = [&](int r, int c, int idx) {
        if (idx == word.size()) return true;
        if (r < 0 || r >= m || c < 0 || c >= n) return false;
        if (board[r][c] != word[idx]) return false;

        char temp = board[r][c];
        board[r][c] = '#';  // Mark visited

        for (auto [dr, dc] : dirs) {
            if (backtrack(r + dr, c + dc, idx + 1)) {
                return true;
            }
        }

        board[r][c] = temp;  // Restore
        return false;
    };

    for (int i = 0; i < m; ++i) {
        for (int j = 0; j < n; ++j) {
            if (backtrack(i, j, 0)) return true;
        }
    }
    return false;
}
```

### Generate Parentheses

```cpp
std::vector<std::string> generateParenthesis(int n) {
    std::vector<std::string> result;
    std::string current;

    std::function<void(int, int)> backtrack = [&](int open, int close) {
        if (current.size() == 2 * n) {
            result.push_back(current);
            return;
        }

        if (open < n) {
            current += '(';
            backtrack(open + 1, close);
            current.pop_back();
        }

        if (close < open) {
            current += ')';
            backtrack(open, close + 1);
            current.pop_back();
        }
    };

    backtrack(0, 0);
    return result;
}
```

## Pruning Techniques

1. **Skip invalid choices early**
2. **Use visited/used arrays**
3. **Calculate remaining elements needed**
4. **Sort to handle duplicates**

```cpp
// Handle duplicates in permutations
void backtrack(vector<int>& nums, vector<bool>& used, vector<int>& current) {
    // ...
    for (int i = 0; i < nums.size(); ++i) {
        if (used[i]) continue;
        // Skip duplicates: same value, previous not used
        if (i > 0 && nums[i] == nums[i-1] && !used[i-1]) continue;
        // ...
    }
}
```

## Key Takeaways

- Try all possibilities, backtrack on failure
- Use recursion with state modification and restoration
- Pruning improves performance significantly
- Common patterns: subsets, permutations, combinations
- Time complexity often exponential (but that's inherent to the problem)

## Common Interview Questions

> [!question]- How is backtracking different from DFS?
> Backtracking is DFS with state management. In backtracking, you explicitly undo choices (backtrack) when exploring different branches.

> [!question]- How do you handle duplicates?
> Sort the input first, then skip consecutive duplicates when the previous duplicate wasn't used in the current branch.
