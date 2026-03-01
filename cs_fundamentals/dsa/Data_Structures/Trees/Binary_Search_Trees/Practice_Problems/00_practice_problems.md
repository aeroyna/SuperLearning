# Practice Problems: Binary Search Trees

Leveraging the BST property (Left < Node < Right).

## Validation & Traversal

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Validate Binary Search Tree](https://leetcode.com/problems/validate-binary-search-tree/) | Medium | Recursive with `(min, max)` range constraints. |
| [Kth Smallest Element in a BST](https://leetcode.com/problems/kth-smallest-element-in-a-bst/) | Medium | In-order traversal gives sorted elements. Stop at k. |
| [Lowest Common Ancestor of a BST](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-search-tree/) | Medium | If both `p,q < root`, go left. If both `> root`, go right. Else `root`. |

## Modification

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Insert into a Binary Search Tree](https://leetcode.com/problems/insert-into-a-binary-search-tree/) | Medium | Traverse to leaf, attach new node. |
| [Delete Node in a BST](https://leetcode.com/problems/delete-node-in-a-bst/) | Medium | Cases: leaf, one child, two children (replace with successor). |
