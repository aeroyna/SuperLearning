## The Backtracking Template

Backtracking is a refined form of recursion used to explore all possible solutions to a problem and prune paths that cannot lead to a valid solution. While problems vary, most backtracking solutions can be implemented using a standard, reusable template.

### Core Idea: Choose, Explore, Unchoose
Backtracking can be visualized as a search through a state-space tree. The goal is to explore paths from the root to the leaves, where leaves represent complete solutions. The template embodies a "choose, explore, unchoose" pattern at each node.

1.  **Choose**: Make a choice from a set of possibilities to move to the next state.
2.  **Explore**: Recursively call the backtracking function on the new state. This moves one level deeper into the search tree.
3.  **Unchoose**: After the recursive call returns (meaning that path has been fully explored), undo the choice you made. This is the "backtracking" step, which allows you to return to the previous state and explore a different path.

### The Generic Template
This template can be adapted for a wide range of problems, such as generating subsets, permutations, or combinations.

>[!example]- C++
>```cpp
>void backtrack(State& state, /* other params */) {
>    // 1. Base Case: Check if the current state is a valid solution
>    if (is_a_solution(state)) {
>        add_solution(state);
>        return;
>    }
>
>    // (Optional) Pruning: Check if the current state can't possibly lead to a solution
>    if (cannot_lead_to_solution(state)) {
>        return;
>    }
>
>    // 2. Iterate through all possible "next" choices
>    for (auto choice : get_all_possible_choices(state)) {
>
>        // 3. Choose: Make the choice
>        make_choice(state, choice);
>
>        // 4. Explore: Recurse with the new state
>        backtrack(new_state, /* other params */);
>
>        // 5. Unchoose: Backtrack to explore other paths
>        undo_choice(state, choice);
>    }
>}
>```

>[!example]- Java
>```java
>void backtrack(State state, /* other params */) {
>    // 1. Base Case: Check if the current state is a valid solution
>    if (is_a_solution(state)) {
>        add_solution(state);
>        return;
>    }
>
>    // (Optional) Pruning: Check if the current state can't possibly lead to a solution
>    if (cannot_lead_to_solution(state)) {
>        return;
>    }
>
>    // 2. Iterate through all possible "next" choices
>    for (Choice choice : get_all_possible_choices(state)) {
>
>        // 3. Choose: Make the choice
>        make_choice(state, choice);
>
>        // 4. Explore: Recurse with the new state
>        backtrack(new_state, /* other params */);
>
>        // 5. Unchoose: Backtrack to explore other paths
>        undo_choice(state, choice);
>    }
>}
>```

>[!example]- Python
>```python
>def backtrack(state, ...other_params):
>    # 1. Base Case: Check if the current state is a valid solution
>    if is_a_solution(state):
>        add_solution(state)
>        return
>
>    # (Optional) Pruning: Check if the current state can't possibly lead to a solution
>    if cannot_lead_to_solution(state):
>        return
>
>    # 2. Iterate through all possible "next" choices
>    for choice in get_all_possible_choices(state):
>
>        # 3. Choose: Make the choice
>        make_choice(state, choice)
>
>        # 4. Explore: Recurse with the new state
>        backtrack(new_state, ...other_params)
>
>        # 5. Unchoose: Backtrack to explore other paths
>        undo_choice(state, choice)
>
>```

>[!example]- JavaScript
>```javascript
>function backtrack(state, /* other params */) {
>    // 1. Base Case: Check if the current state is a valid solution
>    if (is_a_solution(state)) {
>        add_solution(state);
>        return;
>    }
>
>    // (Optional) Pruning: Check if the current state can't possibly lead to a solution
>    if (cannot_lead_to_solution(state)) {
>        return;
>    }
>
>    // 2. Iterate through all possible "next" choices
>    for (let choice of get_all_possible_choices(state)) {
>
>        // 3. Choose: Make the choice
>        make_choice(state, choice);
>
>        // 4. Explore: Recurse with the new state
>        backtrack(new_state, /* other params */);
>
>        // 5. Unchoose: Backtrack to explore other paths
>        undo_choice(state, choice);
>    }
>}
>```

### Example: Generating All Subsets (aka Powerset)
Let's apply the template to generate all subsets of a given set of numbers `[1, 2, 3]`.

>[!example]- C++
>```cpp
>vector<vector<int>> find_subsets(vector<int>& nums) {
>    vector<vector<int>> result;
>    vector<int> current_subset;
>
>    function<void(int)> backtrack = [&](int start_index) {
>        // 1. Base Case: Every path is a valid subset. Add a copy of the current state.
>        result.push_back(current_subset);
>
>        // 2. Iterate through choices (the remaining numbers)
>        for (int i = start_index; i < nums.size(); i++) {
>
>            // 3. Choose: Add the number to the current subset
>            current_subset.push_back(nums[i]);
>
>            // 4. Explore: Recurse, starting from the next index to avoid duplicates
>            backtrack(i + 1);
>
>            // 5. Unchoose: Backtrack by removing the number
>            current_subset.pop_back();
>        }
>    };
>
>    // Initial call to start the process
>    backtrack(0);
>    return result;
>}
>
>// Calling find_subsets([1, 2, 3]) would produce:
>// [[], [1], [1, 2], [1, 2, 3], [1, 3], [2], [2, 3], [3]]
>```

>[!example]- Java
>```java
>public List<List<Integer>> find_subsets(int[] nums) {
>    List<List<Integer>> result = new ArrayList<>();
>    List<Integer> current_subset = new ArrayList<>();
>
>    backtrack(0, nums, current_subset, result);
>    return result;
>}
>
>private void backtrack(int start_index, int[] nums,
>                       List<Integer> current_subset,
>                       List<List<Integer>> result) {
>    // 1. Base Case: Every path is a valid subset. Add a copy of the current state.
>    result.add(new ArrayList<>(current_subset));
>
>    // 2. Iterate through choices (the remaining numbers)
>    for (int i = start_index; i < nums.length; i++) {
>
>        // 3. Choose: Add the number to the current subset
>        current_subset.add(nums[i]);
>
>        // 4. Explore: Recurse, starting from the next index to avoid duplicates
>        backtrack(i + 1, nums, current_subset, result);
>
>        // 5. Unchoose: Backtrack by removing the number
>        current_subset.remove(current_subset.size() - 1);
>    }
>}
>
>// Calling find_subsets([1, 2, 3]) would produce:
>// [[], [1], [1, 2], [1, 2, 3], [1, 3], [2], [2, 3], [3]]
>```

>[!example]- Python
>```python
>def find_subsets(nums):
>    result = []
>    current_subset = []
>
>    def backtrack(start_index):
>        # 1. Base Case: Every path is a valid subset. Add a copy of the current state.
>        result.append(list(current_subset))
>
>        # 2. Iterate through choices (the remaining numbers)
>        for i in range(start_index, len(nums)):
>
>            # 3. Choose: Add the number to the current subset
>            current_subset.append(nums[i])
>
>            # 4. Explore: Recurse, starting from the next index to avoid duplicates
>            backtrack(i + 1)
>
>            # 5. Unchoose: Backtrack by removing the number
>            current_subset.pop()
>
>    # Initial call to start the process
>    backtrack(0)
>    return result
>
># Calling find_subsets([1, 2, 3]) would produce:
># [[], [1], [1, 2], [1, 2, 3], [1, 3], [2], [2, 3], [3]]
>```

>[!example]- JavaScript
>```javascript
>function find_subsets(nums) {
>    const result = [];
>    const current_subset = [];
>
>    function backtrack(start_index) {
>        // 1. Base Case: Every path is a valid subset. Add a copy of the current state.
>        result.push([...current_subset]);
>
>        // 2. Iterate through choices (the remaining numbers)
>        for (let i = start_index; i < nums.length; i++) {
>
>            // 3. Choose: Add the number to the current subset
>            current_subset.push(nums[i]);
>
>            // 4. Explore: Recurse, starting from the next index to avoid duplicates
>            backtrack(i + 1);
>
>            // 5. Unchoose: Backtrack by removing the number
>            current_subset.pop();
>        }
>    }
>
>    // Initial call to start the process
>    backtrack(0);
>    return result;
>}
>
>// Calling find_subsets([1, 2, 3]) would produce:
>// [[], [1], [1, 2], [1, 2, 3], [1, 3], [2], [2, 3], [3]]
>```

In this example:
- `state` is managed by `current_subset` and `start_index`.
- `is_a_solution` is always true; every combination we build is a valid subset.
- `make_choice` is `current_subset.append()`.
- `undo_choice` is `current_subset.pop()`.

Mastering this template is key to solving a wide variety of combinatorial search problems.
