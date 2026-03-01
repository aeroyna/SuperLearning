## Fenwick Trees (Binary Indexed Trees)

A Fenwick Tree, also known as a Binary Indexed Tree (BIT), is a data structure that provides efficient methods for calculating **prefix sums** and updating element values in an array. For problems that require both range sum queries and point updates, it is often a more space-efficient and easier-to-code alternative to a Segment Tree.

### Core Idea
A Fenwick Tree allows both of the following operations to be performed in **O(log n)** time:
1.  **Point Update**: Update the value of an element at a specific index.
2.  **Prefix Sum Query**: Calculate the sum of elements from the start of the array up to a specific index `i`.

A **Range Sum Query** for an interval `[l, r]` can then be easily calculated in O(log n) time by finding `query(r) - query(l-1)`.

The magic of a Fenwick Tree lies in how it stores information. It uses an array, where each index `i` is "responsible" for storing the sum of a specific range of elements from the original array. The size and start of this range are determined by the binary representation of `i`, specifically its **least significant bit (LSB)**.

### Key Bitwise Operations
The entire structure relies on a few clever bitwise operations on the 1-indexed array indices.
- **Isolating the LSB**: The expression `i & -i` gives you the value of the least significant bit of `i`. For example, for `i = 12 (1100)`, `-i` is `(0100)`, so `1100 & 0100` is `0100`, which is 4. This tells you that index 12 is responsible for a range of 4 elements.
- **Moving to the Parent (for updates)**: To find the next node that needs to be updated, you move to the parent by adding the LSB: `i += (i & -i)`.
- **Moving to the Next Sub-range (for queries)**: To get the next part of a prefix sum, you move to the previous responsible index by subtracting the LSB: `i -= (i & -i)`.

### Implementation (1-Indexed)
A 1-indexed implementation is often more intuitive as the bitwise operations align more cleanly.

>[!example]- C++
>```cpp
>#include <vector>
>using namespace std;
>
>class FenwickTree {
>private:
>    vector<int> tree;
>
>public:
>    FenwickTree(int size) {
>        // The BIT is 1-indexed, so we need size + 1
>        tree.resize(size + 1, 0);
>    }
>
>    /**
>     * Adds 'delta' to the value at 'index'.
>     */
>    void update(int index, int delta) {
>        // Start at the 1-based index
>        index += 1;
>        while (index < tree.size()) {
>            tree[index] += delta;
>            // Move to the next parent index
>            index += (index & -index);
>        }
>    }
>
>    /**
>     * Computes the prefix sum up to 'index' (inclusive).
>     */
>    int query(int index) {
>        // Start at the 1-based index
>        index += 1;
>        int sum = 0;
>        while (index > 0) {
>            sum += tree[index];
>            // Move to the next responsible index
>            index -= (index & -index);
>        }
>        return sum;
>    }
>
>    /**
>     * Computes the sum of elements from 'left' to 'right'.
>     */
>    int queryRange(int left, int right) {
>        return query(right) - (left > 0 ? query(left - 1) : 0);
>    }
>};
>
>// Example Usage:
>// vector<int> nums = {1, 2, 3, 4, 5};
>// FenwickTree bit(nums.size());
>// for (int i = 0; i < nums.size(); i++) {
>//     bit.update(i, nums[i]);
>// }
>//
>// // Get sum of range [1, 3], which is 2 + 3 + 4 = 9
>// cout << bit.queryRange(1, 3) << endl; // Output: 9
>//
>// // Update the value at index 2 from 3 to 5 (delta = 2)
>// bit.update(2, 2);
>//
>// // Query the range again
>// cout << bit.queryRange(1, 3) << endl; // Output: 11 (2 + 5 + 4)
>```

>[!example]- Java
>```java
>class FenwickTree {
>    private int[] tree;
>
>    public FenwickTree(int size) {
>        // The BIT is 1-indexed, so we need size + 1
>        tree = new int[size + 1];
>    }
>
>    /**
>     * Adds 'delta' to the value at 'index'.
>     */
>    public void update(int index, int delta) {
>        // Start at the 1-based index
>        index += 1;
>        while (index < tree.length) {
>            tree[index] += delta;
>            // Move to the next parent index
>            index += (index & -index);
>        }
>    }
>
>    /**
>     * Computes the prefix sum up to 'index' (inclusive).
>     */
>    public int query(int index) {
>        // Start at the 1-based index
>        index += 1;
>        int sum = 0;
>        while (index > 0) {
>            sum += tree[index];
>            // Move to the next responsible index
>            index -= (index & -index);
>        }
>        return sum;
>    }
>
>    /**
>     * Computes the sum of elements from 'left' to 'right'.
>     */
>    public int queryRange(int left, int right) {
>        return query(right) - (left > 0 ? query(left - 1) : 0);
>    }
>}
>
>// Example Usage:
>// int[] nums = {1, 2, 3, 4, 5};
>// FenwickTree bit = new FenwickTree(nums.length);
>// for (int i = 0; i < nums.length; i++) {
>//     bit.update(i, nums[i]);
>// }
>//
>// // Get sum of range [1, 3], which is 2 + 3 + 4 = 9
>// System.out.println(bit.queryRange(1, 3)); // Output: 9
>//
>// // Update the value at index 2 from 3 to 5 (delta = 2)
>// bit.update(2, 2);
>//
>// // Query the range again
>// System.out.println(bit.queryRange(1, 3)); // Output: 11 (2 + 5 + 4)
>```

>[!example]- Python
>```python
>class FenwickTree:
>    def __init__(self, size):
>        # The BIT is 1-indexed, so we need size + 1
>        self.tree = [0] * (size + 1)
>
>    def update(self, index, delta):
>        """Adds 'delta' to the value at 'index'."""
>        # Start at the 1-based index
>        index += 1
>        while index < len(self.tree):
>            self.tree[index] += delta
>            # Move to the next parent index
>            index += (index & -index)
>
>    def query(self, index):
>        """Computes the prefix sum up to 'index' (inclusive)."""
>        # Start at the 1-based index
>        index += 1
>        s = 0
>        while index > 0:
>            s += self.tree[index]
>            # Move to the next responsible index
>            index -= (index & -index)
>        return s
>
>    def query_range(self, left, right):
>        """Computes the sum of elements from 'left' to 'right'."""
>        return self.query(right) - self.query(left - 1)
>
># Example Usage:
>nums = [1, 2, 3, 4, 5]
>bit = FenwickTree(len(nums))
>for i, num in enumerate(nums):
>    bit.update(i, num)
>
># Get sum of range [1, 3], which is 2 + 3 + 4 = 9
>print(bit.query_range(1, 3)) # Output: 9
>
># Update the value at index 2 from 3 to 5 (delta = 2)
>bit.update(2, 2)
>
># Query the range again
>print(bit.query_range(1, 3)) # Output: 11 (2 + 5 + 4)
>```

>[!example]- JavaScript
>```javascript
>class FenwickTree {
>    constructor(size) {
>        // The BIT is 1-indexed, so we need size + 1
>        this.tree = new Array(size + 1).fill(0);
>    }
>
>    /**
>     * Adds 'delta' to the value at 'index'.
>     */
>    update(index, delta) {
>        // Start at the 1-based index
>        index += 1;
>        while (index < this.tree.length) {
>            this.tree[index] += delta;
>            // Move to the next parent index
>            index += (index & -index);
>        }
>    }
>
>    /**
>     * Computes the prefix sum up to 'index' (inclusive).
>     */
>    query(index) {
>        // Start at the 1-based index
>        index += 1;
>        let sum = 0;
>        while (index > 0) {
>            sum += this.tree[index];
>            // Move to the next responsible index
>            index -= (index & -index);
>        }
>        return sum;
>    }
>
>    /**
>     * Computes the sum of elements from 'left' to 'right'.
>     */
>    queryRange(left, right) {
>        return this.query(right) - (left > 0 ? this.query(left - 1) : 0);
>    }
>}
>
>// Example Usage:
>const nums = [1, 2, 3, 4, 5];
>const bit = new FenwickTree(nums.length);
>for (let i = 0; i < nums.length; i++) {
>    bit.update(i, nums[i]);
>}
>
>// Get sum of range [1, 3], which is 2 + 3 + 4 = 9
>console.log(bit.queryRange(1, 3)); // Output: 9
>
>// Update the value at index 2 from 3 to 5 (delta = 2)
>bit.update(2, 2);
>
>// Query the range again
>console.log(bit.queryRange(1, 3)); // Output: 11 (2 + 5 + 4)
>```

Fenwick Trees are a powerful tool for a specific subset of problems and are highly valued in competitive programming and challenging interviews.
