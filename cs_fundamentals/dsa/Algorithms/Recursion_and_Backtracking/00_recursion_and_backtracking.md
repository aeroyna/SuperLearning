# Recursion and Backtracking

Backtracking is a systematic way to search through all possible configurations of a solution space by building solutions incrementally and abandoning paths that fail constraints.

## Overview

Backtracking = Try → Check → Backtrack (undo) → Try next

## Topics

- [15.1 Recursion Fundamentals](01_recursion_fundamentals.md)
- [15.2 Backtracking Template](02_backtracking_template.md)
- [15.3 Generation Problems](03_generation_problems.md)
- [15.4 Constraint Satisfaction](04_constraint_satisfaction.md)
- [15.5 Practice Problems](Practice_Problems/00_practice_problems.md)

## Backtracking Template

>[!example]- C++
>```cpp
>void backtrack(State& state, vector<Result>& result) {
>    // Base case: found a valid solution
>    if (isSolution(state)) {
>        result.push_back(state);
>        return;
>    }
>
>    for (const auto& choice : getChoices(state)) {
>        // Check if choice is valid
>        if (isValid(choice, state)) {
>            // Make choice
>            state.add(choice);
>
>            // Recurse with updated state
>            backtrack(state, result);
>
>            // Undo choice (backtrack)
>            state.removeLast();
>        }
>    }
>}
>```

>[!example]- Java
>```java
>void backtrack(State state, List<State> result) {
>    // Base case: found a valid solution
>    if (isSolution(state)) {
>        result.add(new State(state)); // Copy state
>        return;
>    }
>
>    for (Choice choice : getChoices(state)) {
>        // Check if choice is valid
>        if (isValid(choice, state)) {
>            // Make choice
>            state.add(choice);
>
>            // Recurse with updated state
>            backtrack(state, result);
>
>            // Undo choice (backtrack)
>            state.removeLast();
>        }
>    }
>}
>```

>[!example]- Python
>```python
>def backtrack(state, choices):
>    # Base case: found a valid solution
>    if is_solution(state):
>        result.append(state.copy())
>        return
>
>    for choice in choices:
>        # Check if choice is valid
>        if is_valid(choice, state):
>            # Make choice
>            state.append(choice)
>
>            # Recurse with updated state
>            backtrack(state, remaining_choices)
>
>            # Undo choice (backtrack)
>            state.pop()
>```

>[!example]- JavaScript
>```javascript
>function backtrack(state, choices, result) {
>    // Base case: found a valid solution
>    if (isSolution(state)) {
>        result.push([...state]); // Copy state
>        return;
>    }
>
>    for (const choice of choices) {
>        // Check if choice is valid
>        if (isValid(choice, state)) {
>            // Make choice
>            state.push(choice);
>
>            // Recurse with updated state
>            backtrack(state, remainingChoices, result);
>
>            // Undo choice (backtrack)
>            state.pop();
>        }
>    }
>}
>```

## Generation Problems

### Subsets (All Combinations)

>[!example]- C++
>```cpp
>void backtrack(vector<int>& nums, int start, vector<int>& current, vector<vector<int>>& result) {
>    result.push_back(current);
>
>    for (int i = start; i < nums.size(); i++) {
>        current.push_back(nums[i]);
>        backtrack(nums, i + 1, current, result);
>        current.pop_back();
>    }
>}
>
>vector<vector<int>> subsets(vector<int>& nums) {
>    vector<vector<int>> result;
>    vector<int> current;
>    backtrack(nums, 0, current, result);
>    return result;
>}
>```

>[!example]- Java
>```java
>public List<List<Integer>> subsets(int[] nums) {
>    List<List<Integer>> result = new ArrayList<>();
>    backtrack(nums, 0, new ArrayList<>(), result);
>    return result;
>}
>
>private void backtrack(int[] nums, int start, List<Integer> current, List<List<Integer>> result) {
>    result.add(new ArrayList<>(current));
>
>    for (int i = start; i < nums.length; i++) {
>        current.add(nums[i]);
>        backtrack(nums, i + 1, current, result);
>        current.remove(current.size() - 1);
>    }
>}
>```

>[!example]- Python
>```python
>def subsets(nums):
>    result = []
>
>    def backtrack(start, current):
>        result.append(current[:])
>
>        for i in range(start, len(nums)):
>            current.append(nums[i])
>            backtrack(i + 1, current)
>            current.pop()
>
>    backtrack(0, [])
>    return result
>
># Input: [1,2,3]
># Output: [[],[1],[1,2],[1,2,3],[1,3],[2],[2,3],[3]]
>```

>[!example]- JavaScript
>```javascript
>function subsets(nums) {
>    const result = [];
>
>    function backtrack(start, current) {
>        result.push([...current]);
>
>        for (let i = start; i < nums.length; i++) {
>            current.push(nums[i]);
>            backtrack(i + 1, current);
>            current.pop();
>        }
>    }
>
>    backtrack(0, []);
>    return result;
>}
>```

### Permutations

>[!example]- C++
>```cpp
>void backtrack(vector<int>& nums, vector<int>& current, vector<bool>& used, vector<vector<int>>& result) {
>    if (current.size() == nums.size()) {
>        result.push_back(current);
>        return;
>    }
>
>    for (int i = 0; i < nums.size(); i++) {
>        if (used[i]) continue;
>        used[i] = true;
>        current.push_back(nums[i]);
>        backtrack(nums, current, used, result);
>        current.pop_back();
>        used[i] = false;
>    }
>}
>
>vector<vector<int>> permutations(vector<int>& nums) {
>    vector<vector<int>> result;
>    vector<int> current;
>    vector<bool> used(nums.size(), false);
>    backtrack(nums, current, used, result);
>    return result;
>}
>```

>[!example]- Java
>```java
>public List<List<Integer>> permutations(int[] nums) {
>    List<List<Integer>> result = new ArrayList<>();
>    backtrack(nums, new ArrayList<>(), new boolean[nums.length], result);
>    return result;
>}
>
>private void backtrack(int[] nums, List<Integer> current, boolean[] used, List<List<Integer>> result) {
>    if (current.size() == nums.length) {
>        result.add(new ArrayList<>(current));
>        return;
>    }
>
>    for (int i = 0; i < nums.length; i++) {
>        if (used[i]) continue;
>        used[i] = true;
>        current.add(nums[i]);
>        backtrack(nums, current, used, result);
>        current.remove(current.size() - 1);
>        used[i] = false;
>    }
>}
>```

>[!example]- Python
>```python
>def permutations(nums):
>    result = []
>
>    def backtrack(current, remaining):
>        if not remaining:
>            result.append(current[:])
>            return
>
>        for i in range(len(remaining)):
>            current.append(remaining[i])
>            backtrack(current, remaining[:i] + remaining[i+1:])
>            current.pop()
>
>    backtrack([], nums)
>    return result
>```

>[!example]- JavaScript
>```javascript
>function permutations(nums) {
>    const result = [];
>
>    function backtrack(current, remaining) {
>        if (remaining.length === 0) {
>            result.push([...current]);
>            return;
>        }
>
>        for (let i = 0; i < remaining.length; i++) {
>            current.push(remaining[i]);
>            backtrack(current, remaining.slice(0, i).concat(remaining.slice(i + 1)));
>            current.pop();
>        }
>    }
>
>    backtrack([], nums);
>    return result;
>}
>```

### Combinations (C(n,k))

```python
def combinations(n, k):
    result = []

    def backtrack(start, current):
        if len(current) == k:
            result.append(current[:])
            return

        for i in range(start, n + 1):
            current.append(i)
            backtrack(i + 1, current)
            current.pop()

    backtrack(1, [])
    return result
```

## Constraint Satisfaction

### N-Queens

```python
def solveNQueens(n):
    result = []
    cols = set()
    diag1 = set()  # row - col
    diag2 = set()  # row + col

    def backtrack(row, queens):
        if row == n:
            result.append(queens[:])
            return

        for col in range(n):
            if col in cols or (row - col) in diag1 or (row + col) in diag2:
                continue

            cols.add(col)
            diag1.add(row - col)
            diag2.add(row + col)
            queens.append(col)

            backtrack(row + 1, queens)

            queens.pop()
            cols.remove(col)
            diag1.remove(row - col)
            diag2.remove(row + col)

    backtrack(0, [])
    return result
```

### Sudoku Solver

```python
def solveSudoku(board):
    def is_valid(row, col, num):
        # Check row, column, and 3x3 box
        for i in range(9):
            if board[row][i] == num:
                return False
            if board[i][col] == num:
                return False
            box_row = 3 * (row // 3) + i // 3
            box_col = 3 * (col // 3) + i % 3
            if board[box_row][box_col] == num:
                return False
        return True

    def solve():
        for i in range(9):
            for j in range(9):
                if board[i][j] == '.':
                    for num in '123456789':
                        if is_valid(i, j, num):
                            board[i][j] = num
                            if solve():
                                return True
                            board[i][j] = '.'
                    return False
        return True

    solve()
```

## Handling Duplicates

For inputs with duplicates, sort first and skip duplicates:

```python
def subsetsWithDup(nums):
    nums.sort()  # Sort first!
    result = []

    def backtrack(start, current):
        result.append(current[:])

        for i in range(start, len(nums)):
            # Skip duplicates
            if i > start and nums[i] == nums[i-1]:
                continue

            current.append(nums[i])
            backtrack(i + 1, current)
            current.pop()

    backtrack(0, [])
    return result
```

## Complexity

- **Time**: O(b^d) where b = branching factor, d = depth
- **Space**: O(d) for recursion stack

Typical complexities:
- Subsets: O(n × 2^n)
- Permutations: O(n × n!)
- Combinations: O(k × C(n,k))

## Key Interview Problems

| Problem | Type | Difficulty | LeetCode Link |
| --------- | ------ | ------------ | --- |
| Subsets | Generation | Medium | [Link](https://leetcode.com/problems/subsets/) |
| Permutations | Generation | Medium | [Link](https://leetcode.com/problems/permutations/) |
| Combinations | Generation | Medium | [Link](https://leetcode.com/problems/combinations/) |
| Letter Combinations | Generation | Medium | [Link](https://leetcode.com/problems/letter-combinations-of-a-phone-number/) |
| N-Queens | Constraint | Hard | [Link](https://leetcode.com/problems/n-queens/) |
| Sudoku Solver | Constraint | Hard | [Link](https://leetcode.com/problems/sudoku-solver/) |
| Word Search | Grid | Medium | [Link](https://leetcode.com/problems/word-search/) |
| Combination Sum | Constraint | Medium | [Link](https://leetcode.com/problems/combination-sum/) |
