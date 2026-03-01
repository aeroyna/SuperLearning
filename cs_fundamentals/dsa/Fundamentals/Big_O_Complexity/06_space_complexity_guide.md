## A Guide to Space Complexity

Space complexity measures the total amount of memory an algorithm requires in relation to the size of its input (`n`). While time complexity is often the primary focus, understanding space complexity is also crucial, especially for problems involving very large datasets or constrained environments.

Space complexity is composed of two parts:
1.  **Input Space**: The space required to store the input data itself. This is usually not counted against your algorithm unless the problem requires you to modify the input or store it in a different structure.
2.  **Auxiliary Space**: The extra space or temporary memory used by the algorithm during its execution. **This is what we typically mean when we talk about an algorithm's space complexity.**

---

### Common Space Complexities

#### O(1) - Constant Space
The algorithm uses a fixed amount of extra memory, regardless of the input size. This is the most space-efficient category.
- **Examples**:
  - Simple loops with a few variables (`i`, `sum`, `max_val`).
  - In-place algorithms like Heap Sort or algorithms that modify the input array directly by swapping elements.
  - Two Pointers technique on an existing array.

#### O(log n) - Logarithmic Space
The extra space grows logarithmically with the input size. This is very efficient and is most commonly associated with the **recursion call stack**.
- **Examples**:
  - A recursive algorithm on a **balanced** binary tree. The maximum depth of the call stack will be the height of the tree, which is O(log n).
  - Quick Sort's average case space complexity is O(log n) due to its recursive calls. The worst case is O(n) if the tree is unbalanced.

#### O(n) - Linear Space
The extra space grows linearly with the input size. This is very common and often acceptable.
- **Examples**:
  - Creating a copy of the input array or list.
  - A **hash map or hash set** that stores up to `n` unique elements.
  - The recursion call stack for a recursive algorithm on a skewed/unbalanced data structure (like a linked list or a skewed tree), where the recursion depth can be up to `n`.
  - Storing a `visited` set for a graph traversal with `n` nodes.
  - A queue or stack that, in the worst case, holds all `n` elements.

#### O(n^2) - Quadratic Space
The extra space grows quadratically with the input size. This is generally only acceptable for small inputs.
- **Examples**:
  - Creating an adjacency matrix for a graph with `n` vertices.
  - A 2D DP table (`dp[n][n]`) for problems involving pairs of elements from the same input.

### Analyzing Space Complexity

1.  **Variables**: Sum up the space used by all variables declared by the algorithm. Primitives like integers and booleans are constant space. For objects, arrays, and other data structures, the space depends on their size.
2.  **Data Structures**: The dominant factor is often the space used by auxiliary data structures. If you create a hash map, a heap, or another array, its maximum potential size determines its contribution to the space complexity.
3.  **Recursion**: For recursive algorithms, the space complexity is determined by the **maximum depth of the recursion call stack**. It's the longest path of nested function calls that can occur during the execution of the algorithm.

**Example: Recursive Factorial**

>[!example]- C++
>```cpp
>int factorial(int n) {
>    if (n == 0) {
>        return 1;
>    }
>    return n * factorial(n - 1);
>}
>```

>[!example]- Java
>```java
>public int factorial(int n) {
>    if (n == 0) {
>        return 1;
>    }
>    return n * factorial(n - 1);
>}
>```

>[!example]- Python
>```python
>def factorial(n):
>    if n == 0:
>        return 1
>    return n * factorial(n-1)
>```

>[!example]- JavaScript
>```javascript
>function factorial(n) {
>    if (n === 0) {
>        return 1;
>    }
>    return n * factorial(n - 1);
>}
>```

- **Space Complexity**: O(n). The call stack will grow to a depth of `n` (e.g., `factorial(n)` calls `factorial(n-1)`, ... calls `factorial(0)`).
