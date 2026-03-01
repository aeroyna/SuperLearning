## Common Binary Tree Problems

Many binary tree interview questions are variations of a few classic problems. Understanding the patterns behind these core problems will allow you to solve a wide range of tree-related challenges.

### 1. Finding Maximum Depth (or Height)
**Problem**: Given the `root` of a binary tree, find its maximum depth (the number of nodes along the longest path from the root down to the farthest leaf node). (LeetCode #104)

**Pattern**: This is a classic application of **post-order DFS traversal**. The logic is recursive:
1.  **Base Case**: The depth of a null node is 0.
2.  **Recursive Step**: The depth of any other node is `1 + max(depth of left child, depth of right child)`.
The post-order approach ensures that we have the computed depths of the left and right children before we compute the depth of the current node.

>[!example]- C++
>```cpp
>int maxDepth(TreeNode* root) {
>    if (!root) {
>        return 0;
>    }
>
>    int leftDepth = maxDepth(root->left);
>    int rightDepth = maxDepth(root->right);
>
>    return 1 + max(leftDepth, rightDepth);
>}
>```

>[!example]- Java
>```java
>public int maxDepth(TreeNode root) {
>    if (root == null) {
>        return 0;
>    }
>
>    int leftDepth = maxDepth(root.left);
>    int rightDepth = maxDepth(root.right);
>
>    return 1 + Math.max(leftDepth, rightDepth);
>}
>```

>[!example]- Python
>```python
>def max_depth(root):
>    if not root:
>        return 0
>
>    left_depth = max_depth(root.left)
>    right_depth = max_depth(root.right)
>
>    return 1 + max(left_depth, right_depth)
>```

>[!example]- JavaScript
>```javascript
>function maxDepth(root) {
>    if (!root) {
>        return 0;
>    }
>
>    const leftDepth = maxDepth(root.left);
>    const rightDepth = maxDepth(root.right);
>
>    return 1 + Math.max(leftDepth, rightDepth);
>}
>```

### 2. Checking for Same Tree
**Problem**: Given the roots of two binary trees, `p` and `q`, write a function to check if they are the same or not. Two binary trees are considered the same if they are structurally identical, and the nodes have the same value. (LeetCode #100)

**Pattern**: This problem uses a simultaneous **pre-order DFS traversal** on both trees.
1.  **Base Cases**:
    - If both nodes are `null`, they are the same. Return `True`.
    - If one node is `null` and the other is not, they are different. Return `False`.
    - If the node values are different, the trees are different. Return `False`.
2.  **Recursive Step**: If the current nodes are valid, recursively check if the left subtrees are the same AND the right subtrees are the same.

>[!example]- C++
>```cpp
>bool isSameTree(TreeNode* p, TreeNode* q) {
>    // Base cases
>    if (!p && !q) {
>        return true;
>    }
>    if (!p || !q || p->val != q->val) {
>        return false;
>    }
>
>    // Recursive step
>    return isSameTree(p->left, q->left) && isSameTree(p->right, q->right);
>}
>```

>[!example]- Java
>```java
>public boolean isSameTree(TreeNode p, TreeNode q) {
>    // Base cases
>    if (p == null && q == null) {
>        return true;
>    }
>    if (p == null || q == null || p.val != q.val) {
>        return false;
>    }
>
>    // Recursive step
>    return isSameTree(p.left, q.left) && isSameTree(p.right, q.right);
>}
>```

>[!example]- Python
>```python
>def is_same_tree(p, q):
>    # Base cases
>    if not p and not q:
>        return True
>    if not p or not q or p.val != q.val:
>        return False
>
>    # Recursive step
>    return is_same_tree(p.left, q.left) and is_same_tree(p.right, q.right)
>```

>[!example]- JavaScript
>```javascript
>function isSameTree(p, q) {
>    // Base cases
>    if (!p && !q) {
>        return true;
>    }
>    if (!p || !q || p.val !== q.val) {
>        return false;
>    }
>
>    // Recursive step
>    return isSameTree(p.left, q.left) && isSameTree(p.right, q.right);
>}
>```

### 3. Lowest Common Ancestor (LCA)
**Problem**: Given a binary tree, find the lowest common ancestor (LCA) of two given nodes in the tree. The LCA is defined between two nodes `p` and `q` as the lowest node in the tree that has both `p` and `q` as descendants. (LeetCode #236)

**Pattern**: This is a more advanced problem that uses a clever **post-order DFS** traversal. The function returns the node that it finds (`p`, `q`, or the LCA).
1.  **Base Case**: If the current node is `null`, `p`, or `q`, return the current node.
2.  **Recursive Step**:
    - Recursively search for `p` and `q` in the left and right subtrees. Let the results be `left_found` and `right_found`.
    - **Logic**:
        - If both `left_found` and `right_found` are non-null, it means `p` and `q` were found in different subtrees. Therefore, the **current node** is the LCA.
        - If only `left_found` is non-null, it means both `p` and `q` are in the left subtree. Return `left_found`.
        - If only `right_found` is non-null, return `right_found`.

>[!example]- C++
>```cpp
>TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
>    if (!root || root == p || root == q) {
>        return root;
>    }
>
>    TreeNode* leftFound = lowestCommonAncestor(root->left, p, q);
>    TreeNode* rightFound = lowestCommonAncestor(root->right, p, q);
>
>    // If p and q are in different subtrees, root is the LCA
>    if (leftFound && rightFound) {
>        return root;
>    }
>
>    // Otherwise, the LCA is in the subtree that contains both nodes
>    return leftFound ? leftFound : rightFound;
>}
>```

>[!example]- Java
>```java
>public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
>    if (root == null || root == p || root == q) {
>        return root;
>    }
>
>    TreeNode leftFound = lowestCommonAncestor(root.left, p, q);
>    TreeNode rightFound = lowestCommonAncestor(root.right, p, q);
>
>    // If p and q are in different subtrees, root is the LCA
>    if (leftFound != null && rightFound != null) {
>        return root;
>    }
>
>    // Otherwise, the LCA is in the subtree that contains both nodes
>    return leftFound != null ? leftFound : rightFound;
>}
>```

>[!example]- Python
>```python
>def lowest_common_ancestor(root, p, q):
>    if not root or root == p or root == q:
>        return root
>
>    left_found = lowest_common_ancestor(root.left, p, q)
>    right_found = lowest_common_ancestor(root.right, p, q)
>
>    # If p and q are in different subtrees, root is the LCA
>    if left_found and right_found:
>        return root
>
>    # Otherwise, the LCA is in the subtree that contains both nodes
>    return left_found or right_found
>```

>[!example]- JavaScript
>```javascript
>function lowestCommonAncestor(root, p, q) {
>    if (!root || root === p || root === q) {
>        return root;
>    }
>
>    const leftFound = lowestCommonAncestor(root.left, p, q);
>    const rightFound = lowestCommonAncestor(root.right, p, q);
>
>    // If p and q are in different subtrees, root is the LCA
>    if (leftFound && rightFound) {
>        return root;
>    }
>
>    // Otherwise, the LCA is in the subtree that contains both nodes
>    return leftFound || rightFound;
>}
>```

This elegant solution efficiently finds the LCA by propagating the findings up the recursion stack.
