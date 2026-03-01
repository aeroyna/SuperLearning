## Segment Trees

A Segment Tree is a powerful, flexible tree data structure designed for efficiently answering **range queries** on an array. It also allows for fast updates to the array's elements. For problems that repeatedly ask for the sum, minimum, maximum, or another aggregate function over a specific subarray, a segment tree is often the optimal solution.

### Core Idea
A segment tree is a binary tree where each node represents an interval, or "segment," of the original array.
- The **root** of the tree represents the entire array (e.g., interval `[0, n-1]`).
- Each **leaf** node represents a single element of the array.
- Each **internal node** represents the union of the intervals of its children. The value stored in an internal node is an aggregate of the values of its children (e.g., their sum, max, etc.).

For an array `A`, a node representing the interval `[L, R]` would store `sum(A[L...R])`. Its left child would represent the interval `[L, mid]` and its right child would represent `[mid+1, R]`, where `mid = (L + R) // 2`.

### Key Operations
Both querying and updating a segment tree take **O(log n)** time, which is a significant improvement over the O(n) time required for naive range queries.

#### 1. Build
The tree is constructed recursively from the bottom up.
- You start with the entire array range `[0, n-1]`.
- Recursively split the range in half until you reach individual elements (leaf nodes).
- As the recursion unwinds, calculate the value of each internal node by combining the values of its children.
- Building the entire tree takes **O(n)** time.

#### 2. Range Query
To find the sum of a range `[i, j]`, you traverse the tree.
- At any node representing an interval `[L, R]`:
  - If `[L, R]` is completely contained within your query range `[i, j]`, you can use this node's pre-computed value directly.
  - If `[L, R]` partially overlaps with `[i, j]`, you recurse on one or both children.
  - If `[L, R]` is completely outside `[i, j]`, you can ignore this path.
- Because you are traversing a balanced binary tree, this operation takes **O(log n)** time.

#### 3. Point Update
To update the value of an element at a specific index `idx`:
- Traverse the tree to find the leaf node corresponding to `idx`. This takes O(log n) time.
- Update the value of the leaf node.
- As you return from the recursion, update the values of all parent nodes on the path back to the root. This also takes O(log n) time.

### Implementation (Range Sum Example)

>[!example]- C++
>```cpp
>#include <vector>
>using namespace std;
>
>class SegmentTree {
>private:
>    int n;
>    vector<int> tree;
>    vector<int> nums;
>
>    void build(int nodeIdx, int start, int end) {
>        if (start == end) {
>            tree[nodeIdx] = nums[start];
>            return;
>        }
>        int mid = (start + end) / 2;
>        build(2 * nodeIdx + 1, start, mid);
>        build(2 * nodeIdx + 2, mid + 1, end);
>        tree[nodeIdx] = tree[2 * nodeIdx + 1] + tree[2 * nodeIdx + 2];
>    }
>
>    void updateRecursive(int nodeIdx, int start, int end, int idx, int val) {
>        if (start == end) {
>            tree[nodeIdx] = val;
>            return;
>        }
>        int mid = (start + end) / 2;
>        if (start <= idx && idx <= mid) {
>            updateRecursive(2 * nodeIdx + 1, start, mid, idx, val);
>        } else {
>            updateRecursive(2 * nodeIdx + 2, mid + 1, end, idx, val);
>        }
>        tree[nodeIdx] = tree[2 * nodeIdx + 1] + tree[2 * nodeIdx + 2];
>    }
>
>    int queryRecursive(int nodeIdx, int start, int end, int l, int r) {
>        if (r < start || end < l) {
>            return 0; // Range is outside
>        }
>        if (l <= start && end <= r) {
>            return tree[nodeIdx]; // Range is completely inside
>        }
>        int mid = (start + end) / 2;
>        int p1 = queryRecursive(2 * nodeIdx + 1, start, mid, l, r);
>        int p2 = queryRecursive(2 * nodeIdx + 2, mid + 1, end, l, r);
>        return p1 + p2;
>    }
>
>public:
>    SegmentTree(vector<int>& nums) {
>        this->n = nums.size();
>        this->nums = nums;
>        tree.resize(4 * n); // A safe upper bound for tree size
>        build(0, 0, n - 1);
>    }
>
>    void update(int idx, int val) {
>        updateRecursive(0, 0, n - 1, idx, val);
>    }
>
>    int query(int l, int r) {
>        return queryRecursive(0, 0, n - 1, l, r);
>    }
>};
>```

>[!example]- Java
>```java
>class SegmentTree {
>    private int n;
>    private int[] tree;
>    private int[] nums;
>
>    public SegmentTree(int[] nums) {
>        this.n = nums.length;
>        this.nums = nums;
>        this.tree = new int[4 * n]; // A safe upper bound for tree size
>        build(0, 0, n - 1);
>    }
>
>    private void build(int nodeIdx, int start, int end) {
>        if (start == end) {
>            tree[nodeIdx] = nums[start];
>            return;
>        }
>        int mid = (start + end) / 2;
>        build(2 * nodeIdx + 1, start, mid);
>        build(2 * nodeIdx + 2, mid + 1, end);
>        tree[nodeIdx] = tree[2 * nodeIdx + 1] + tree[2 * nodeIdx + 2];
>    }
>
>    public void update(int idx, int val) {
>        updateRecursive(0, 0, n - 1, idx, val);
>    }
>
>    private void updateRecursive(int nodeIdx, int start, int end, int idx, int val) {
>        if (start == end) {
>            tree[nodeIdx] = val;
>            return;
>        }
>        int mid = (start + end) / 2;
>        if (start <= idx && idx <= mid) {
>            updateRecursive(2 * nodeIdx + 1, start, mid, idx, val);
>        } else {
>            updateRecursive(2 * nodeIdx + 2, mid + 1, end, idx, val);
>        }
>        tree[nodeIdx] = tree[2 * nodeIdx + 1] + tree[2 * nodeIdx + 2];
>    }
>
>    public int query(int l, int r) {
>        return queryRecursive(0, 0, n - 1, l, r);
>    }
>
>    private int queryRecursive(int nodeIdx, int start, int end, int l, int r) {
>        if (r < start || end < l) {
>            return 0; // Range is outside
>        }
>        if (l <= start && end <= r) {
>            return tree[nodeIdx]; // Range is completely inside
>        }
>        int mid = (start + end) / 2;
>        int p1 = queryRecursive(2 * nodeIdx + 1, start, mid, l, r);
>        int p2 = queryRecursive(2 * nodeIdx + 2, mid + 1, end, l, r);
>        return p1 + p2;
>    }
>}
>```

>[!example]- Python
>```python
>class SegmentTree:
>    def __init__(self, nums):
>        self.n = len(nums)
>        self.tree = [0] * (4 * self.n) # A safe upper bound for tree size
>        self.nums = nums
>        self.build(0, 0, self.n - 1)
>
>    def build(self, node_idx, start, end):
>        if start == end:
>            self.tree[node_idx] = self.nums[start]
>            return
>        mid = (start + end) // 2
>        self.build(2 * node_idx + 1, start, mid)
>        self.build(2 * node_idx + 2, mid + 1, end)
>        self.tree[node_idx] = self.tree[2 * node_idx + 1] + self.tree[2 * node_idx + 2]
>
>    def update(self, idx, val):
>        self._update_recursive(0, 0, self.n - 1, idx, val)
>
>    def _update_recursive(self, node_idx, start, end, idx, val):
>        if start == end:
>            self.tree[node_idx] = val
>            return
>        mid = (start + end) // 2
>        if start <= idx <= mid:
>            self._update_recursive(2 * node_idx + 1, start, mid, idx, val)
>        else:
>            self._update_recursive(2 * node_idx + 2, mid + 1, end, idx, val)
>        self.tree[node_idx] = self.tree[2 * node_idx + 1] + self.tree[2 * node_idx + 2]
>
>    def query(self, l, r):
>        return self._query_recursive(0, 0, self.n - 1, l, r)
>
>    def _query_recursive(self, node_idx, start, end, l, r):
>        if r < start or end < l:
>            return 0 # Range is outside
>        if l <= start and end <= r:
>            return self.tree[node_idx] # Range is completely inside
>        mid = (start + end) // 2
>        p1 = self._query_recursive(2 * node_idx + 1, start, mid, l, r)
>        p2 = self._query_recursive(2 * node_idx + 2, mid + 1, end, l, r)
>        return p1 + p2
>```

>[!example]- JavaScript
>```javascript
>class SegmentTree {
>    constructor(nums) {
>        this.n = nums.length;
>        this.tree = new Array(4 * this.n).fill(0); // A safe upper bound for tree size
>        this.nums = nums;
>        this.build(0, 0, this.n - 1);
>    }
>
>    build(nodeIdx, start, end) {
>        if (start === end) {
>            this.tree[nodeIdx] = this.nums[start];
>            return;
>        }
>        const mid = Math.floor((start + end) / 2);
>        this.build(2 * nodeIdx + 1, start, mid);
>        this.build(2 * nodeIdx + 2, mid + 1, end);
>        this.tree[nodeIdx] = this.tree[2 * nodeIdx + 1] + this.tree[2 * nodeIdx + 2];
>    }
>
>    update(idx, val) {
>        this.updateRecursive(0, 0, this.n - 1, idx, val);
>    }
>
>    updateRecursive(nodeIdx, start, end, idx, val) {
>        if (start === end) {
>            this.tree[nodeIdx] = val;
>            return;
>        }
>        const mid = Math.floor((start + end) / 2);
>        if (start <= idx && idx <= mid) {
>            this.updateRecursive(2 * nodeIdx + 1, start, mid, idx, val);
>        } else {
>            this.updateRecursive(2 * nodeIdx + 2, mid + 1, end, idx, val);
>        }
>        this.tree[nodeIdx] = this.tree[2 * nodeIdx + 1] + this.tree[2 * nodeIdx + 2];
>    }
>
>    query(l, r) {
>        return this.queryRecursive(0, 0, this.n - 1, l, r);
>    }
>
>    queryRecursive(nodeIdx, start, end, l, r) {
>        if (r < start || end < l) {
>            return 0; // Range is outside
>        }
>        if (l <= start && end <= r) {
>            return this.tree[nodeIdx]; // Range is completely inside
>        }
>        const mid = Math.floor((start + end) / 2);
>        const p1 = this.queryRecursive(2 * nodeIdx + 1, start, mid, l, r);
>        const p2 = this.queryRecursive(2 * nodeIdx + 2, mid + 1, end, l, r);
>        return p1 + p2;
>    }
>}
>```

While complex to implement, segment trees are a very powerful tool for a specific class of range-based problems.
