## Bitmasks

A bitmask is a powerful technique where the bits of an integer are used to represent a set of boolean flags. Instead of using a boolean array or a hash set to keep track of the state of multiple items (e.g., visited nodes, items in a subset), you can use the bits of a single integer. This is extremely space-efficient and can lead to very fast operations.

### The Core Idea
An integer is stored in memory as a sequence of bits (usually 32 or 64). We can assign a meaning to each of these bits. For a problem with `N` items, we can use an `N`-bit integer where the `i`-th bit corresponds to the `i`-th item.

- If the `i`-th bit is `1`, it means the `i`-th item is "on", "visited", or "included".
- If the `i`-th bit is `0`, it means the `i`-th item is "off", "not visited", or "not included".

For example, with `N=4` items, the integer `13` (binary `1101`) could represent the set `{item 0, item 2, item 3}`.

### When to Use Bitmasks
This technique is particularly useful for problems where `N` (the number of items) is small, typically up to around 20, because the number of possible subsets is `2^N`.
- **Generating Subsets**: You can iterate from `0` to `2^N - 1`. Each integer `i` in this range represents a unique subset. You can then check which bits are set in `i` to determine which elements are in that subset.
- **Dynamic Programming with Bitmasking (DP on Subsets)**: This is an advanced DP technique for solving problems like the Traveling Salesperson Problem (TSP) on a small number of nodes. The state of the DP can be `dp(mask, last_node)`, where `mask` represents the set of visited nodes.
- **State Representation**: Storing a compact representation of a visited state in a graph traversal or backtracking problem, where the mask can be used as a key in a memoization table.

### Bitmask Operations
You can use the common bitwise tricks to manipulate the set represented by the mask.

Let `mask` be the integer representing the set.
- **Add an item `i` to the set**: Set the `i`-th bit.
  `mask = mask | (1 << i)`
- **Remove an item `i` from the set**: Clear the `i`-th bit.
  `mask = mask & ~(1 << i)`
- **Check if item `i` is in the set**: Check if the `i`-th bit is set.
  `is_present = (mask & (1 << i)) != 0`
- **Toggle item `i`'s presence**: Flip the `i`-th bit.
  `mask = mask ^ (1 << i)`

### Example: Generating All Subsets

>[!example]- C++
>```cpp
>vector<vector<int>> generateSubsets(vector<int>& nums) {
>    int n = nums.size();
>    vector<vector<int>> allSubsets;
>
>    // Iterate through all possible bitmasks from 0 to 2^n - 1
>    for (int i = 0; i < (1 << n); i++) {
>        vector<int> subset;
>        // Check each bit of the mask 'i'
>        for (int j = 0; j < n; j++) {
>            // If the j-th bit is set in the mask 'i'
>            if ((i & (1 << j)) != 0) {
>                subset.push_back(nums[j]);
>            }
>        }
>        allSubsets.push_back(subset);
>    }
>
>    return allSubsets;
>}
>
>// Example: nums = [1, 2, 3]
>// n = 3
>// The loop for i will go from 0 (000) to 7 (111)
>// When i = 5 (101), the 0th and 2nd bits are set.
>// This corresponds to the subset [nums[0], nums[2]], which is [1, 3].
>```

>[!example]- Java
>```java
>public List<List<Integer>> generateSubsets(int[] nums) {
>    int n = nums.length;
>    List<List<Integer>> allSubsets = new ArrayList<>();
>
>    // Iterate through all possible bitmasks from 0 to 2^n - 1
>    for (int i = 0; i < (1 << n); i++) {
>        List<Integer> subset = new ArrayList<>();
>        // Check each bit of the mask 'i'
>        for (int j = 0; j < n; j++) {
>            // If the j-th bit is set in the mask 'i'
>            if ((i & (1 << j)) != 0) {
>                subset.add(nums[j]);
>            }
>        }
>        allSubsets.add(subset);
>    }
>
>    return allSubsets;
>}
>
>// Example: nums = [1, 2, 3]
>// n = 3
>// The loop for i will go from 0 (000) to 7 (111)
>// When i = 5 (101), the 0th and 2nd bits are set.
>// This corresponds to the subset [nums[0], nums[2]], which is [1, 3].
>```

>[!example]- Python
>```python
>def generate_subsets(nums):
>    n = len(nums)
>    all_subsets = []
>
>    # Iterate through all possible bitmasks from 0 to 2^n - 1
>    for i in range(1 << n):
>        subset = []
>        # Check each bit of the mask 'i'
>        for j in range(n):
>            # If the j-th bit is set in the mask 'i'
>            if (i & (1 << j)) != 0:
>                subset.append(nums[j])
>        all_subsets.append(subset)
>
>    return all_subsets
>
># Example: nums = [1, 2, 3]
># n = 3
># The loop for i will go from 0 (000) to 7 (111)
># When i = 5 (101), the 0th and 2nd bits are set.
># This corresponds to the subset [nums[0], nums[2]], which is [1, 3].
>```

>[!example]- JavaScript
>```javascript
>function generateSubsets(nums) {
>    const n = nums.length;
>    const allSubsets = [];
>
>    // Iterate through all possible bitmasks from 0 to 2^n - 1
>    for (let i = 0; i < (1 << n); i++) {
>        const subset = [];
>        // Check each bit of the mask 'i'
>        for (let j = 0; j < n; j++) {
>            // If the j-th bit is set in the mask 'i'
>            if ((i & (1 << j)) !== 0) {
>                subset.push(nums[j]);
>            }
>        }
>        allSubsets.push(subset);
>    }
>
>    return allSubsets;
>}
>
>// Example: nums = [1, 2, 3]
>// n = 3
>// The loop for i will go from 0 (000) to 7 (111)
>// When i = 5 (101), the 0th and 2nd bits are set.
>// This corresponds to the subset [nums[0], nums[2]], which is [1, 3].
>```

This iterative approach is an alternative to the recursive backtracking method for generating subsets and demonstrates the power of using bitmasks to represent sets.
