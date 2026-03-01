# Array Fundamentals

## What is an Array?

An array is a collection of elements stored in contiguous memory locations. Each element can be accessed directly using its index.

## Key Properties

1. **Fixed Size** (static arrays) or **Dynamic** (ArrayList, vector, list)
2. **Contiguous Memory** - Elements stored adjacent in memory
3. **Random Access** - O(1) access to any element by index
4. **Homogeneous** - All elements same type (in statically typed languages)

## Array Operations

### Accessing Elements

>[!example]- C++
>```cpp
>// O(1) - Direct memory access
>#include <vector>
>using namespace std;
>
>vector<int> arr = {10, 20, 30, 40, 50};
>int element = arr[2];  // 30
>```

>[!example]- Java
>```java
>// O(1) - Direct memory access
>int[] arr = {10, 20, 30, 40, 50};
>int element = arr[2];  // 30
>```

>[!example]- Python
>```python
># O(1) - Direct memory access
>arr = [10, 20, 30, 40, 50]
>element = arr[2]  # 30
>```

>[!example]- JavaScript
>```javascript
>// O(1) - Direct memory access
>const arr = [10, 20, 30, 40, 50];
>const element = arr[2];  // 30
>```

### Modifying Elements

>[!example]- C++
>```cpp
>// O(1) - Direct assignment
>arr[2] = 35;  // arr becomes {10, 20, 35, 40, 50}
>```

>[!example]- Java
>```java
>// O(1) - Direct assignment
>arr[2] = 35;  // arr becomes {10, 20, 35, 40, 50}
>```

>[!example]- Python
>```python
># O(1) - Direct assignment
>arr[2] = 35  # [10, 20, 35, 40, 50]
>```

>[!example]- JavaScript
>```javascript
>// O(1) - Direct assignment
>arr[2] = 35;  // [10, 20, 35, 40, 50]
>```

### Inserting Elements

>[!example]- C++
>```cpp
>// O(1) amortized - At end
>arr.push_back(60);  // {10, 20, 35, 40, 50, 60}
>
>// O(n) - At beginning or middle (shifting required)
>arr.insert(arr.begin(), 5);  // {5, 10, 20, 35, 40, 50, 60}
>```

>[!example]- Java
>```java
>import java.util.ArrayList;
>
>// O(1) amortized - At end
>ArrayList<Integer> arr = new ArrayList<>();
>arr.add(60);  // [10, 20, 35, 40, 50, 60]
>
>// O(n) - At beginning or middle (shifting required)
>arr.add(0, 5);  // [5, 10, 20, 35, 40, 50, 60]
>```

>[!example]- Python
>```python
># O(1) amortized - At end
>arr.append(60)  # [10, 20, 35, 40, 50, 60]
>
># O(n) - At beginning or middle (shifting required)
>arr.insert(0, 5)  # [5, 10, 20, 35, 40, 50, 60]
>```

>[!example]- JavaScript
>```javascript
>// O(1) amortized - At end
>arr.push(60);  // [10, 20, 35, 40, 50, 60]
>
>// O(n) - At beginning or middle (shifting required)
>arr.unshift(5);  // [5, 10, 20, 35, 40, 50, 60]
>// Or using splice: arr.splice(0, 0, 5);
>```

### Deleting Elements

>[!example]- C++
>```cpp
>// O(n) - Requires shifting
>arr.erase(arr.begin());  // Remove first: O(n)
>arr.pop_back();          // Remove last: O(1)
>
>// Remove by value: O(n)
>arr.erase(remove(arr.begin(), arr.end(), 35), arr.end());
>```

>[!example]- Java
>```java
>// O(n) - Requires shifting
>arr.remove(0);    // Remove first: O(n)
>arr.remove(arr.size() - 1);  // Remove last: O(1)
>
>// Remove by value: O(n)
>arr.remove(Integer.valueOf(35));
>```

>[!example]- Python
>```python
># O(n) - Requires shifting
>arr.pop(0)    # Remove first: O(n)
>arr.pop()     # Remove last: O(1)
>arr.remove(35)  # Remove by value: O(n)
>```

>[!example]- JavaScript
>```javascript
>// O(n) - Requires shifting
>arr.shift();     // Remove first: O(n)
>arr.pop();       // Remove last: O(1)
>
>// Remove by value: O(n)
>const index = arr.indexOf(35);
>if (index > -1) arr.splice(index, 1);
>```

### Searching

>[!example]- C++
>```cpp
>#include <algorithm>
>#include <vector>
>
>// O(n) - Linear search (unsorted)
>auto it = find(arr.begin(), arr.end(), 40);
>int index = distance(arr.begin(), it);
>
>// O(log n) - Binary search (sorted)
>vector<int> sorted_arr = {10, 20, 30, 40, 50};
>auto it2 = lower_bound(sorted_arr.begin(), sorted_arr.end(), target);
>int index2 = distance(sorted_arr.begin(), it2);
>```

>[!example]- Java
>```java
>import java.util.Arrays;
>
>// O(n) - Linear search (unsorted)
>int index = arr.indexOf(40);  // 3
>
>// O(log n) - Binary search (sorted)
>int[] sortedArr = {10, 20, 30, 40, 50};
>int index2 = Arrays.binarySearch(sortedArr, target);
>```

>[!example]- Python
>```python
># O(n) - Linear search (unsorted)
>index = arr.index(40)  # 3
>
># O(log n) - Binary search (sorted)
>import bisect
>index = bisect.bisect_left(sorted_arr, target)
>```

>[!example]- JavaScript
>```javascript
>// O(n) - Linear search (unsorted)
>const index = arr.indexOf(40);  // 3
>
>// O(log n) - Binary search (sorted) - Custom implementation
>function binarySearch(arr, target) {
>    let left = 0, right = arr.length - 1;
>    while (left <= right) {
>        const mid = Math.floor((left + right) / 2);
>        if (arr[mid] === target) return mid;
>        if (arr[mid] < target) left = mid + 1;
>        else right = mid - 1;
>    }
>    return left;
>}
>```

## Common Array Patterns

### 1. Traversal

>[!example]- C++
>```cpp
>// Forward traversal
>for (int i = 0; i < arr.size(); i++) {
>    cout << arr[i] << endl;
>}
>
>// Backward traversal
>for (int i = arr.size() - 1; i >= 0; i--) {
>    cout << arr[i] << endl;
>}
>
>// With index and value (range-based loop)
>for (int i = 0; i < arr.size(); i++) {
>    cout << i << " " << arr[i] << endl;
>}
>```

>[!example]- Java
>```java
>// Forward traversal
>for (int i = 0; i < arr.length; i++) {
>    System.out.println(arr[i]);
>}
>
>// Backward traversal
>for (int i = arr.length - 1; i >= 0; i--) {
>    System.out.println(arr[i]);
>}
>
>// Enhanced for loop with index
>for (int i = 0; i < arr.length; i++) {
>    System.out.println(i + " " + arr[i]);
>}
>```

>[!example]- Python
>```python
># Forward traversal
>for i in range(len(arr)):
>    print(arr[i])
>
># Backward traversal
>for i in range(len(arr) - 1, -1, -1):
>    print(arr[i])
>
># With index and value
>for i, val in enumerate(arr):
>    print(i, val)
>```

>[!example]- JavaScript
>```javascript
>// Forward traversal
>for (let i = 0; i < arr.length; i++) {
>    console.log(arr[i]);
>}
>
>// Backward traversal
>for (let i = arr.length - 1; i >= 0; i--) {
>    console.log(arr[i]);
>}
>
>// With index and value
>arr.forEach((val, i) => {
>    console.log(i, val);
>});
>```

### 2. In-Place Modification

>[!example]- C++
>```cpp
>// Remove duplicates from sorted array
>int removeDuplicates(vector<int>& nums) {
>    if (nums.empty()) return 0;
>
>    int write = 1;
>    for (int read = 1; read < nums.size(); read++) {
>        if (nums[read] != nums[read - 1]) {
>            nums[write] = nums[read];
>            write++;
>        }
>    }
>    return write;
>}
>```

>[!example]- Java
>```java
>// Remove duplicates from sorted array
>public int removeDuplicates(int[] nums) {
>    if (nums.length == 0) return 0;
>
>    int write = 1;
>    for (int read = 1; read < nums.length; read++) {
>        if (nums[read] != nums[read - 1]) {
>            nums[write] = nums[read];
>            write++;
>        }
>    }
>    return write;
>}
>```

>[!example]- Python
>```python
># Remove duplicates from sorted array
>def removeDuplicates(nums):
>    if not nums:
>        return 0
>    write = 1
>    for read in range(1, len(nums)):
>        if nums[read] != nums[read - 1]:
>            nums[write] = nums[read]
>            write += 1
>    return write
>```

>[!example]- JavaScript
>```javascript
>// Remove duplicates from sorted array
>function removeDuplicates(nums) {
>    if (nums.length === 0) return 0;
>
>    let write = 1;
>    for (let read = 1; read < nums.length; read++) {
>        if (nums[read] !== nums[read - 1]) {
>            nums[write] = nums[read];
>            write++;
>        }
>    }
>    return write;
>}
>```

### 3. Building Result Array

>[!example]- C++
>```cpp
>#include <vector>
>using namespace std;
>
>// Transform each element
>vector<int> result;
>for (int x : arr) {
>    result.push_back(x * 2);
>}
>
>// Filter elements
>vector<int> filtered;
>for (int x : arr) {
>    if (x > 0) {
>        filtered.push_back(x);
>    }
>}
>```

>[!example]- Java
>```java
>import java.util.ArrayList;
>import java.util.stream.Collectors;
>
>// Transform each element
>List<Integer> result = arr.stream()
>    .map(x -> x * 2)
>    .collect(Collectors.toList());
>
>// Filter elements
>List<Integer> filtered = arr.stream()
>    .filter(x -> x > 0)
>    .collect(Collectors.toList());
>```

>[!example]- Python
>```python
># Transform each element
>result = [x * 2 for x in arr]
>
># Filter elements
>result = [x for x in arr if x > 0]
>```

>[!example]- JavaScript
>```javascript
>// Transform each element
>const result = arr.map(x => x * 2);
>
>// Filter elements
>const filtered = arr.filter(x => x > 0);
>```

## Subarray Concepts

A **subarray** is a contiguous portion of an array.

```
Array: [1, 2, 3, 4]

Subarrays of length 1: [1], [2], [3], [4]
Subarrays of length 2: [1,2], [2,3], [3,4]
Subarrays of length 3: [1,2,3], [2,3,4]
Subarrays of length 4: [1,2,3,4]

Total subarrays = n(n+1)/2
```

## Subsequence vs Subarray

- **Subarray**: Contiguous elements in original order
- **Subsequence**: Not necessarily contiguous, but in original order

```
Array: [1, 2, 3, 4]

[2, 3] - Both subarray and subsequence
[1, 3] - Subsequence only (not contiguous)
[3, 1] - Neither (wrong order)
```

## 2D Arrays (Matrices)

>[!example]- C++
>```cpp
>#include <vector>
>using namespace std;
>
>// Create 2D array
>int rows = 3, cols = 4;
>vector<vector<int>> matrix(rows, vector<int>(cols, 0));
>
>// Access element
>int element = matrix[row][col];
>
>// Common traversals
>// Row by row
>for (const auto& row : matrix) {
>    for (int element : row) {
>        cout << element << " ";
>    }
>}
>
>// Column by column
>for (int col = 0; col < matrix[0].size(); col++) {
>    for (int row = 0; row < matrix.size(); row++) {
>        cout << matrix[row][col] << " ";
>    }
>}
>
>// Diagonal
>for (int i = 0; i < min(rows, cols); i++) {
>    cout << matrix[i][i] << " ";
>}
>```

>[!example]- Java
>```java
>// Create 2D array
>int rows = 3, cols = 4;
>int[][] matrix = new int[rows][cols];
>
>// Access element
>int element = matrix[row][col];
>
>// Common traversals
>// Row by row
>for (int[] row : matrix) {
>    for (int element : row) {
>        System.out.print(element + " ");
>    }
>}
>
>// Column by column
>for (int col = 0; col < matrix[0].length; col++) {
>    for (int row = 0; row < matrix.length; row++) {
>        System.out.print(matrix[row][col] + " ");
>    }
>}
>
>// Diagonal
>for (int i = 0; i < Math.min(rows, cols); i++) {
>    System.out.print(matrix[i][i] + " ");
>}
>```

>[!example]- Python
>```python
># Create 2D array
>matrix = [[0] * cols for _ in range(rows)]
>
># Access element
>element = matrix[row][col]
>
># Common traversals
># Row by row
>for row in matrix:
>    for element in row:
>        print(element)
>
># Column by column
>for col in range(len(matrix[0])):
>    for row in range(len(matrix)):
>        print(matrix[row][col])
>
># Diagonal
>for i in range(min(rows, cols)):
>    print(matrix[i][i])
>```

>[!example]- JavaScript
>```javascript
>// Create 2D array
>const rows = 3, cols = 4;
>const matrix = Array.from({length: rows}, () => Array(cols).fill(0));
>
>// Access element
>const element = matrix[row][col];
>
>// Common traversals
>// Row by row
>for (const row of matrix) {
>    for (const element of row) {
>        console.log(element);
>    }
>}
>
>// Column by column
>for (let col = 0; col < matrix[0].length; col++) {
>    for (let row = 0; row < matrix.length; row++) {
>        console.log(matrix[row][col]);
>    }
>}
>
>// Diagonal
>for (let i = 0; i < Math.min(rows, cols); i++) {
>    console.log(matrix[i][i]);
>}
>```

## Memory Considerations

### Static vs Dynamic Arrays

| Aspect | Static Array | Dynamic Array |
|--------|--------------|---------------|
| Size | Fixed at creation | Can grow/shrink |
| Memory | Allocated once | May reallocate |
| Insert at end | N/A or O(n) | O(1) amortized |
| Memory overhead | Minimal | Extra capacity |

### Cache Performance

Arrays have excellent cache performance because:
1. Elements are stored contiguously
2. Predictable memory access patterns
3. CPU can prefetch data efficiently

This makes array iteration very fast compared to linked structures.

## Common Interview Problems

1. **Find maximum/minimum** - Single pass O(n)
2. **Find duplicates** - Hash set or sorting
3. **Rotate array** - Multiple approaches
4. **Merge intervals** - Sort + iterate
5. **Product except self** - Prefix/suffix products
