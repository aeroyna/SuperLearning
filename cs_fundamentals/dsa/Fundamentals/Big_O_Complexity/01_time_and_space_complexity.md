# Time and Space Complexity

## Time Complexity by Data Structure

### Arrays (Dynamic Array/List)

Given `n = arr.length`:

| Operation | Time Complexity | Notes |
|-----------|-----------------|-------|
| Access/modify at index | O(1) | Direct memory access |
| Add/remove at end | O(1) amortized | May need resize |
| Add/remove at arbitrary index | O(n) | Shifting required |
| Check if element exists | O(n) | Linear search |
| Two pointers / Sliding window | O(n * k) | k = work per iteration |
| Building prefix sum | O(n) | Single pass |
| Query prefix sum | O(1) | After preprocessing |

### Strings (Immutable)

Given `n = s.length`:

| Operation | Time Complexity | Notes |
|-----------|-----------------|-------|
| Access character at index | O(1) | Direct access |
| Add/remove character | O(n) | Creates new string |
| Concatenation | O(n + m) | m = other string length |
| Create substring | O(m) | m = substring length |
| Build string from array/StringBuilder | O(n) | |

### Linked Lists

Given `n` = number of nodes:

| Operation | Time Complexity | Notes |
|-----------|-----------------|-------|
| Add/remove with pointer to location | O(1) | No traversal needed |
| Add/remove at arbitrary position | O(n) | Traversal required |
| Access at arbitrary position | O(n) | No random access |
| Check if element exists | O(n) | Linear search |
| Reverse between positions i and j | O(j - i) | |
| Detect cycle | O(n) | Fast-slow pointers |

### Hash Table / Dictionary

Given `n = dic.length`:

| Operation | Time Complexity | Notes |
|-----------|-----------------|-------|
| Add/remove key-value pair | O(1) average | O(n) worst case |
| Check if key exists | O(1) average | |
| Check if value exists | O(n) | Must scan all values |
| Access/modify value by key | O(1) average | |
| Iterate over all entries | O(n) | |

> **Note**: O(1) is relative to hash table size. If keys are strings, hashing costs O(m) where m = string length.

### Set

Given `n = set.length`:

| Operation | Time Complexity |
|-----------|-----------------|
| Add/remove element | O(1) average |
| Check if element exists | O(1) average |

### Stack

Given `n = stack.length` (implemented with dynamic array):

| Operation | Time Complexity |
|-----------|-----------------|
| Push element | O(1) |
| Pop element | O(1) |
| Peek top element | O(1) |
| Check if element exists | O(n) |

### Queue

Given `n = queue.length` (implemented with doubly linked list):

| Operation | Time Complexity |
|-----------|-----------------|
| Enqueue element | O(1) |
| Dequeue element | O(1) |
| Peek front element | O(1) |
| Check if element exists | O(n) |

### Binary Tree (DFS/BFS)

Given `n` = number of nodes:

Most algorithms run in **O(n * k)** where k = work done at each node (usually O(1)).

### Binary Search Tree

Given `n` = number of nodes:

| Operation | Average Case | Worst Case |
|-----------|--------------|------------|
| Add/remove element | O(log n) | O(n) |
| Search for element | O(log n) | O(n) |

> Worst case occurs when tree is unbalanced (essentially a linked list).

### Heap / Priority Queue

Given `n = heap.length`:

| Operation | Time Complexity |
|-----------|-----------------|
| Add element | O(log n) |
| Remove min/max | O(log n) |
| Find min/max | O(1) |
| Check if element exists | O(n) |
| Build heap from array | O(n) |

### Trie

Given `n` = number of words, `m` = average word length:

| Operation | Time Complexity |
|-----------|-----------------|
| Insert word | O(m) |
| Search word | O(m) |
| Search prefix | O(m) |

## Algorithm Complexities

| Algorithm | Time Complexity | Space Complexity |
|-----------|-----------------|------------------|
| Binary Search | O(log n) | O(1) iterative, O(log n) recursive |
| Sorting (comparison-based) | O(n log n) | Varies |
| DFS/BFS on graph | O(V + E) | O(V) |
| Dynamic Programming | O(states * work per state) | O(states) |
| Backtracking | O(branches^depth) | O(depth) |

## Space Complexity Considerations

### Common Space Patterns

1. **O(1)** - Fixed number of variables
2. **O(n)** - Array/hash map proportional to input
3. **O(n^2)** - 2D matrix/grid
4. **O(log n)** - Recursive call stack (binary search)
5. **O(n)** - Recursive call stack (linear recursion)

### Hidden Space Costs

>[!example]- C++
>```cpp
>// Sorting in-place: O(1) extra space
>std::sort(arr.begin(), arr.end());
>
>// Sorting creates new vector: O(n) space
>std::vector<int> sorted_arr = arr;
>std::sort(sorted_arr.begin(), sorted_arr.end());
>
>// String concatenation in loop: O(n^2) total!
>std::string result = "";
>for (char c : chars) {
>    result += c;  // Creates new string each time
>}
>
>// Better: O(n) with pre-allocation or string stream
>std::string result;
>result.reserve(chars.size());
>for (char c : chars) {
>    result += c;
>}
>// Or use stringstream
>std::stringstream ss;
>for (char c : chars) {
>    ss << c;
>}
>std::string result = ss.str();
>```

>[!example]- Java
>```java
>// Sorting in-place: O(1) extra space (excluding stack space)
>Arrays.sort(arr);
>
>// Sorting creates new array: O(n) space
>int[] sortedArr = arr.clone();
>Arrays.sort(sortedArr);
>
>// String concatenation in loop: O(n^2) total!
>String result = "";
>for (char c : chars) {
>    result += c;  // Creates new string each time
>}
>
>// Better: O(n) with StringBuilder
>StringBuilder result = new StringBuilder();
>for (char c : chars) {
>    result.append(c);
>}
>return result.toString();
>```

>[!example]- Python
>```python
># Sorting in-place: O(1) extra space
>arr.sort()
>
># Sorting creates new array: O(n) space
>sorted_arr = sorted(arr)
>
># String concatenation in loop: O(n^2) total!
>result = ""
>for char in chars:
>    result += char  # Creates new string each time
>
># Better: O(n) with list
>result = []
>for char in chars:
>    result.append(char)
>return "".join(result)
>```

>[!example]- JavaScript
>```javascript
>// Sorting in-place: O(1) extra space
>arr.sort((a, b) => a - b);
>
>// Sorting creates new array: O(n) space
>const sortedArr = [...arr].sort((a, b) => a - b);
>
>// String concatenation in loop: O(n^2) total!
>let result = "";
>for (const char of chars) {
>    result += char;  // Creates new string each time
>}
>
>// Better: O(n) with array join
>const result = chars.join("");
>// Or use array and join
>const resultArr = [];
>for (const char of chars) {
>    resultArr.push(char);
>}
>return resultArr.join("");
>```

## Amortized Analysis

Some operations have different average vs worst-case complexity:

| Operation | Amortized | Worst Case | Example |
|-----------|-----------|------------|---------|
| Dynamic array append | O(1) | O(n) | Occasional resize |
| Hash table insert | O(1) | O(n) | Rehashing |

**Key Insight**: Expensive operations happen rarely enough that the average cost remains low.
