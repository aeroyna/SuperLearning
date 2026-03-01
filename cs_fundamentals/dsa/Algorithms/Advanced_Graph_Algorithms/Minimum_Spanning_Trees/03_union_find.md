## The Union-Find Data Structure (Disjoint-Set)

The Union-Find data structure (also known as a Disjoint-Set Union or DSU) is a data structure that tracks a set of elements partitioned into a number of disjoint (non-overlapping) subsets. It is a critical component for efficiently implementing Kruskal's algorithm for finding a Minimum Spanning Tree.

### The Core Idea
Imagine you have a set of items, and you want to group them together. The Union-Find structure provides two primary operations:

1.  **`find(i)`**: Determine which subset an element `i` belongs to. This is used to check if two elements are in the same subset. The `find` operation returns a "representative" or "root" item that is a unique identifier for that subset.
2.  **`union(i, j)`**: Merge the two subsets containing elements `i` and `j` into a single subset.

This is exactly what Kruskal's algorithm needs: a way to check if two vertices are already connected (i.e., in the same component/subset) before adding an edge between them, and a way to merge their components if they are not.

### Implementation
A Union-Find structure is typically implemented using an array, `parent`, where `parent[i]` stores the parent of element `i`. The elements of a subset form a tree, and the root of the tree is the representative of that subset. An element is the root of its own tree if it is its own parent (`parent[i] == i`).

### Key Optimizations
A naive implementation can be inefficient. Two key optimizations are critical for achieving the near-constant time complexity that makes Union-Find so powerful.

1.  **Path Compression**: During a `find(i)` operation, after finding the root of the set, we can make all nodes on the path from `i` to the root point directly to the root. This flattens the tree, making future `find` operations much faster for those nodes.

2.  **Union by Rank (or Size)**: When performing a `union` operation, instead of arbitrarily making one tree a child of the other, we attach the smaller tree to the root of the larger tree. This helps to keep the trees from becoming too deep. "Rank" usually refers to the height of the tree, while "size" refers to the number of elements.

When both optimizations are used, the time complexity of the operations becomes **nearly constant** on average (amortized), specifically O(α(n)), where α(n) is the extremely slow-growing inverse Ackermann function.

### Implementation

>[!example]- C++
>```cpp
>#include <vector>
>using namespace std;
>
>class UnionFind {
>public:
>    vector<int> parent;
>    vector<int> rank;
>
>    UnionFind(int size) {
>        parent.resize(size);
>        rank.resize(size, 1);
>        for (int i = 0; i < size; i++) {
>            parent[i] = i;
>        }
>    }
>
>    int find(int i) {
>        // Finds the representative of the set containing i, with path compression
>        if (parent[i] == i) {
>            return i;
>        }
>        // Path compression: set parent directly to the root
>        parent[i] = find(parent[i]);
>        return parent[i];
>    }
>
>    bool unite(int i, int j) {
>        // Merges the sets containing i and j, using union by rank
>        int rootI = find(i);
>        int rootJ = find(j);
>
>        if (rootI != rootJ) {
>            // Union by rank: attach smaller tree to the root of the larger tree
>            if (rank[rootI] > rank[rootJ]) {
>                parent[rootJ] = rootI;
>            } else if (rank[rootI] < rank[rootJ]) {
>                parent[rootI] = rootJ;
>            } else {
>                // If ranks are same, make one a child and increment the rank
>                parent[rootJ] = rootI;
>                rank[rootI]++;
>            }
>            return true; // The two were in different sets
>        }
>
>        return false; // The two were already in the same set
>    }
>};
>```

>[!example]- Java
>```java
>public class UnionFind {
>    private int[] parent;
>    private int[] rank;
>
>    public UnionFind(int size) {
>        parent = new int[size];
>        rank = new int[size];
>        for (int i = 0; i < size; i++) {
>            parent[i] = i;
>            rank[i] = 1;
>        }
>    }
>
>    public int find(int i) {
>        // Finds the representative of the set containing i, with path compression
>        if (parent[i] == i) {
>            return i;
>        }
>        // Path compression: set parent directly to the root
>        parent[i] = find(parent[i]);
>        return parent[i];
>    }
>
>    public boolean union(int i, int j) {
>        // Merges the sets containing i and j, using union by rank
>        int rootI = find(i);
>        int rootJ = find(j);
>
>        if (rootI != rootJ) {
>            // Union by rank: attach smaller tree to the root of the larger tree
>            if (rank[rootI] > rank[rootJ]) {
>                parent[rootJ] = rootI;
>            } else if (rank[rootI] < rank[rootJ]) {
>                parent[rootI] = rootJ;
>            } else {
>                // If ranks are same, make one a child and increment the rank
>                parent[rootJ] = rootI;
>                rank[rootI]++;
>            }
>            return true; // The two were in different sets
>        }
>
>        return false; // The two were already in the same set
>    }
>}
>```

>[!example]- Python
>```python
>class UnionFind:
>    def __init__(self, size):
>        # Initialize each node to be its own parent
>        self.parent = list(range(size))
>        # Optional: Initialize ranks for the union-by-rank optimization
>        self.rank = [1] * size
>
>    def find(self, i):
>        """Finds the representative of the set containing i, with path compression."""
>        if self.parent[i] == i:
>            return i
>        # Path compression: set parent directly to the root
>        self.parent[i] = self.find(self.parent[i])
>        return self.parent[i]
>
>    def union(self, i, j):
>        """Merges the sets containing i and j, using union by rank."""
>        root_i = self.find(i)
>        root_j = self.find(j)
>
>        if root_i != root_j:
>            # Union by rank: attach smaller tree to the root of the larger tree
>            if self.rank[root_i] > self.rank[root_j]:
>                self.parent[root_j] = root_i
>            elif self.rank[root_i] < self.rank[root_j]:
>                self.parent[root_i] = root_j
>            else:
>                # If ranks are same, make one a child and increment the rank
>                self.parent[root_j] = root_i
>                self.rank[root_i] += 1
>            return True # The two were in different sets
>
>        return False # The two were already in the same set
>```

>[!example]- JavaScript
>```javascript
>class UnionFind {
>    constructor(size) {
>        // Initialize each node to be its own parent
>        this.parent = Array(size).fill(0).map((_, i) => i);
>        // Initialize ranks for the union-by-rank optimization
>        this.rank = Array(size).fill(1);
>    }
>
>    find(i) {
>        // Finds the representative of the set containing i, with path compression
>        if (this.parent[i] === i) {
>            return i;
>        }
>        // Path compression: set parent directly to the root
>        this.parent[i] = this.find(this.parent[i]);
>        return this.parent[i];
>    }
>
>    union(i, j) {
>        // Merges the sets containing i and j, using union by rank
>        const rootI = this.find(i);
>        const rootJ = this.find(j);
>
>        if (rootI !== rootJ) {
>            // Union by rank: attach smaller tree to the root of the larger tree
>            if (this.rank[rootI] > this.rank[rootJ]) {
>                this.parent[rootJ] = rootI;
>            } else if (this.rank[rootI] < this.rank[rootJ]) {
>                this.parent[rootI] = rootJ;
>            } else {
>                // If ranks are same, make one a child and increment the rank
>                this.parent[rootJ] = rootI;
>                this.rank[rootI]++;
>            }
>            return true; // The two were in different sets
>        }
>
>        return false; // The two were already in the same set
>    }
>}
>```

This data structure is not only for MSTs but is also useful for any problem involving connected components, such as checking for cycles in an undirected graph or network connectivity problems.
