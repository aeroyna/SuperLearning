# Practice Problems: Binary Trees

General tree traversal, recursion, and properties.

## Traversal (DFS/BFS)

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Maximum Depth of Binary Tree](https://leetcode.com/problems/maximum-depth-of-binary-tree/) | Easy | `1 + max(left, right)`. |
| [Binary Tree Level Order Traversal](https://leetcode.com/problems/binary-tree-level-order-traversal/) | Medium | BFS using Queue. |
| [Binary Tree Right Side View](https://leetcode.com/problems/binary-tree-right-side-view/) | Medium | BFS (last in level) or DFS (Root->Right->Left). |

## Recursion

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Invert Binary Tree](https://leetcode.com/problems/invert-binary-tree/) | Easy | Swap left/right, recurse. |
| [Same Tree](https://leetcode.com/problems/same-tree/) | Easy | Check val, recurse left/right. Base cases: both null, one null. |
| [Diameter of Binary Tree](https://leetcode.com/problems/diameter-of-binary-tree/) | Easy | `L+R` at each node. Return `1 + max(L, R)` height. |
| [Lowest Common Ancestor of a Binary Tree](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree/) | Medium | If `root` in `{p,q}`, return `root`. If left & right return non-null, root is LCA. |
| [Serialize and Deserialize Binary Tree](https://leetcode.com/problems/serialize-and-deserialize-binary-tree/) | Hard | Preorder (Root, L, R) with "null" markers. |
