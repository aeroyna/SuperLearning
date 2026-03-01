## Backtracking: Generation Problems

A major category of backtracking problems involves **generating** all possible combinatorial objects that meet a certain criteria. This includes finding all possible subsets, permutations, or combinations of a given set of elements.

The backtracking template is a perfect fit for these problems, as it naturally explores the entire search space of possibilities. The key is to correctly define the "choices" at each step of the recursion.

### 1. Generating Subsets (The Powerset)
- **Problem**: Given a set of elements, find all of its possible subsets.
- **Choices at each step**: For each element in the set, you have two choices: either **include** it in the current subset or **do not include** it.
- **Implementation**: The template shown in `02_backtracking_template.md` is a classic way to generate subsets. You iterate through the elements, and at each step, you decide whether to add the current element and recurse, then backtrack and skip the current element.

### 2. Generating Permutations
- **Problem**: Given a collection of distinct integers, return all possible permutations (rearrangements) of the numbers.
- **Choices at each step**: At each step of building a permutation, your choice is which of the *remaining, unused* numbers to add next.
- **Implementation**: To manage the "unused" numbers, you can pass a boolean `used` array in your recursive calls or remove elements from a list of available candidates.

>[!example]- C++
>```cpp
>vector<vector<int>> find_permutations(vector<int>& nums) {
>    vector<vector<int>> result;
>    vector<int> current_permutation;
>    vector<bool> used(nums.size(), false);
>
>    function<void()> backtrack = [&]() {
>        // Base Case: If the current permutation is the same size as the input,
>        // we have a complete and valid solution.
>        if (current_permutation.size() == nums.size()) {
>            result.push_back(current_permutation);
>            return;
>        }
>
>        // Iterate through all numbers to make a choice
>        for (int i = 0; i < nums.size(); i++) {
>            // Pruning: Only choose numbers that have not been used yet
>            if (used[i]) {
>                continue;
>            }
>
>            // Choose
>            used[i] = true;
>            current_permutation.push_back(nums[i]);
>
>            // Explore
>            backtrack();
>
>            // Unchoose (backtrack)
>            current_permutation.pop_back();
>            used[i] = false;
>        }
>    };
>
>    backtrack();
>    return result;
>}
>```

>[!example]- Java
>```java
>public List<List<Integer>> find_permutations(int[] nums) {
>    List<List<Integer>> result = new ArrayList<>();
>    List<Integer> current_permutation = new ArrayList<>();
>    boolean[] used = new boolean[nums.length];
>
>    backtrack(nums, current_permutation, used, result);
>    return result;
>}
>
>private void backtrack(int[] nums, List<Integer> current_permutation,
>                       boolean[] used, List<List<Integer>> result) {
>    // Base Case: If the current permutation is the same size as the input,
>    // we have a complete and valid solution.
>    if (current_permutation.size() == nums.length) {
>        result.add(new ArrayList<>(current_permutation));
>        return;
>    }
>
>    // Iterate through all numbers to make a choice
>    for (int i = 0; i < nums.length; i++) {
>        // Pruning: Only choose numbers that have not been used yet
>        if (used[i]) {
>            continue;
>        }
>
>        // Choose
>        used[i] = true;
>        current_permutation.add(nums[i]);
>
>        // Explore
>        backtrack(nums, current_permutation, used, result);
>
>        // Unchoose (backtrack)
>        current_permutation.remove(current_permutation.size() - 1);
>        used[i] = false;
>    }
>}
>```

>[!example]- Python
>```python
>def find_permutations(nums):
>    result = []
>    current_permutation = []
>    used = [False] * len(nums)
>
>    def backtrack():
>        # Base Case: If the current permutation is the same size as the input,
>        # we have a complete and valid solution.
>        if len(current_permutation) == len(nums):
>            result.append(list(current_permutation))
>            return
>
>        # Iterate through all numbers to make a choice
>        for i in range(len(nums)):
>            # Pruning: Only choose numbers that have not been used yet
>            if used[i]:
>                continue
>
>            # Choose
>            used[i] = True
>            current_permutation.append(nums[i])
>
>            # Explore
>            backtrack()
>
>            # Unchoose (backtrack)
>            current_permutation.pop()
>            used[i] = False
>
>    backtrack()
>    return result
>```

>[!example]- JavaScript
>```javascript
>function find_permutations(nums) {
>    const result = [];
>    const current_permutation = [];
>    const used = new Array(nums.length).fill(false);
>
>    function backtrack() {
>        // Base Case: If the current permutation is the same size as the input,
>        // we have a complete and valid solution.
>        if (current_permutation.length === nums.length) {
>            result.push([...current_permutation]);
>            return;
>        }
>
>        // Iterate through all numbers to make a choice
>        for (let i = 0; i < nums.length; i++) {
>            // Pruning: Only choose numbers that have not been used yet
>            if (used[i]) {
>                continue;
>            }
>
>            // Choose
>            used[i] = true;
>            current_permutation.push(nums[i]);
>
>            // Explore
>            backtrack();
>
>            // Unchoose (backtrack)
>            current_permutation.pop();
>            used[i] = false;
>        }
>    }
>
>    backtrack();
>    return result;
>}
>```

### 3. Generating Combinations
- **Problem**: Given two integers `n` and `k`, return all possible combinations of `k` numbers out of the range `[1, n]`.
- **Choices at each step**: This is very similar to generating subsets, but with a constraint on the final size (`k`). You choose numbers to include in the combination.
- **Implementation**: You can adapt the subset generation template. You pass a `start_index` to avoid generating duplicate combinations (e.g., `[1, 2]` is the same as `[2, 1]`). You add a base case to stop once the current combination has reached the desired size `k`.

>[!example]- C++
>```cpp
>vector<vector<int>> find_combinations(int n, int k) {
>    vector<vector<int>> result;
>    vector<int> current_combination;
>
>    function<void(int)> backtrack = [&](int start_index) {
>        // Base Case: If the combination is the right size, add it.
>        if (current_combination.size() == k) {
>            result.push_back(current_combination);
>            return;
>        }
>
>        // Pruning: If there aren't enough numbers left to fill the combination, stop.
>        if (current_combination.size() + (n - start_index + 1) < k) {
>            return;
>        }
>
>        // Iterate through choices (numbers from start_index to n)
>        for (int i = start_index; i <= n; i++) {
>            // Choose
>            current_combination.push_back(i);
>            // Explore
>            backtrack(i + 1);
>            // Unchoose
>            current_combination.pop_back();
>        }
>    };
>
>    backtrack(1);
>    return result;
>}
>```

>[!example]- Java
>```java
>public List<List<Integer>> find_combinations(int n, int k) {
>    List<List<Integer>> result = new ArrayList<>();
>    List<Integer> current_combination = new ArrayList<>();
>
>    backtrack(1, n, k, current_combination, result);
>    return result;
>}
>
>private void backtrack(int start_index, int n, int k,
>                       List<Integer> current_combination,
>                       List<List<Integer>> result) {
>    // Base Case: If the combination is the right size, add it.
>    if (current_combination.size() == k) {
>        result.add(new ArrayList<>(current_combination));
>        return;
>    }
>
>    // Pruning: If there aren't enough numbers left to fill the combination, stop.
>    if (current_combination.size() + (n - start_index + 1) < k) {
>        return;
>    }
>
>    // Iterate through choices (numbers from start_index to n)
>    for (int i = start_index; i <= n; i++) {
>        // Choose
>        current_combination.add(i);
>        // Explore
>        backtrack(i + 1, n, k, current_combination, result);
>        // Unchoose
>        current_combination.remove(current_combination.size() - 1);
>    }
>}
>```

>[!example]- Python
>```python
>def find_combinations(n, k):
>    result = []
>    current_combination = []
>
>    def backtrack(start_index):
>        # Base Case: If the combination is the right size, add it.
>        if len(current_combination) == k:
>            result.append(list(current_combination))
>            return
>
>        # Pruning: If there aren't enough numbers left to fill the combination, stop.
>        if len(current_combination) + (n - start_index + 1) < k:
>            return
>
>        # Iterate through choices (numbers from start_index to n)
>        for i in range(start_index, n + 1):
>            # Choose
>            current_combination.append(i)
>            # Explore
>            backtrack(i + 1)
>            # Unchoose
>            current_combination.pop()
>
>    backtrack(1)
>    return result
>```

>[!example]- JavaScript
>```javascript
>function find_combinations(n, k) {
>    const result = [];
>    const current_combination = [];
>
>    function backtrack(start_index) {
>        // Base Case: If the combination is the right size, add it.
>        if (current_combination.length === k) {
>            result.push([...current_combination]);
>            return;
>        }
>
>        // Pruning: If there aren't enough numbers left to fill the combination, stop.
>        if (current_combination.length + (n - start_index + 1) < k) {
>            return;
>        }
>
>        // Iterate through choices (numbers from start_index to n)
>        for (let i = start_index; i <= n; i++) {
>            // Choose
>            current_combination.push(i);
>            // Explore
>            backtrack(i + 1);
>            // Unchoose
>            current_combination.pop();
>        }
>    }
>
>    backtrack(1);
>    return result;
>}
>```

These generation problems are fundamental, and mastering their backtracking patterns is essential for many harder interview questions.
