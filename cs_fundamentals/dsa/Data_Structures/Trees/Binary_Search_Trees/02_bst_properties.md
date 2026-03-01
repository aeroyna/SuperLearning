## Key Properties of Binary Search Trees

Beyond the fundamental ordering rule, BSTs have several important properties that are frequently tested in coding interviews.

### 1. In-order Traversal yields a Sorted Sequence
This is the most critical property to remember. When you perform an **in-order traversal** (Left → Node → Right) on a BST, you will visit the nodes in ascending sorted order.

>[!example]- C++
>```cpp
>vector<int> inorderTraversal(TreeNode* node) {
>    if (!node) {
>        return {};
>    }
>    // The vector concatenation demonstrates the order
>    vector<int> result;
>    vector<int> left = inorderTraversal(node->left);
>    result.insert(result.end(), left.begin(), left.end());
>    result.push_back(node->val);
>    vector<int> right = inorderTraversal(node->right);
>    result.insert(result.end(), right.begin(), right.end());
>    return result;
>}
>
>// For a valid BST, the output of this function will be a sorted list.
>```

>[!example]- Java
>```java
>public List<Integer> inorderTraversal(TreeNode node) {
>    if (node == null) {
>        return new ArrayList<>();
>    }
>    // The list concatenation demonstrates the order
>    List<Integer> result = new ArrayList<>();
>    result.addAll(inorderTraversal(node.left));
>    result.add(node.val);
>    result.addAll(inorderTraversal(node.right));
>    return result;
>}
>
>// For a valid BST, the output of this function will be a sorted list.
>```

>[!example]- Python
>```python
>def inorder_traversal(node):
>    if not node:
>        return []
>    # The list concatenation demonstrates the order
>    return inorder_traversal(node.left) + [node.val] + inorder_traversal(node.right)
>
># For a valid BST, the output of this function will be a sorted list.
>```

>[!example]- JavaScript
>```javascript
>function inorderTraversal(node) {
>    if (!node) {
>        return [];
>    }
>    // The array concatenation demonstrates the order
>    return [
>        ...inorderTraversal(node.left),
>        node.val,
>        ...inorderTraversal(node.right)
>    ];
>}
>
>// For a valid BST, the output of this function will be a sorted list.
>```

This property has several direct applications:
- **Validate a BST**: Perform an in-order traversal and check if the resulting sequence is sorted. If it is, the tree is a valid BST. (Note: This is often less efficient than the recursive validation with min/max bounds).
- **Find k-th smallest/largest element**: Perform an in-order traversal and stop at the k-th element.
- **Find the minimum/maximum element**: The minimum element is the leftmost node. The maximum element is the rightmost node.

### 2. Validation of a BST

A common interview problem is to determine if a given binary tree is a valid BST. A naive check that only compares a node with its immediate children (`node.left.val < node.val < node.right.val`) is **incorrect**. This check fails because it doesn't ensure that a node is greater/less than *all* the nodes in its respective subtrees.

**Correct Approach: Recursive Validation with Min/Max Bounds**

The correct way to validate a BST is to perform a traversal (typically DFS) while passing down the valid range (minimum and maximum allowed values) for each node.

1.  Start the recursion at the root with an unbounded range (`-infinity` to `+infinity`).
2.  For any given node, check if its value falls within the `(min_bound, max_bound)` range passed down from its parent. If not, it's an invalid BST.
3.  When recursing on the **left child**, update the `max_bound` to be the current node's value. The left child must be smaller than its parent.
4.  When recursing on the **right child**, update the `min_bound` to be the current node's value. The right child must be larger than its parent.

>[!example]- C++
>```cpp
>bool isValidBST(TreeNode* node, long minBound = LONG_MIN, long maxBound = LONG_MAX) {
>    // An empty tree is a valid BST
>    if (!node) {
>        return true;
>    }
>
>    // Check if the current node's value is within its valid range
>    if (node->val <= minBound || node->val >= maxBound) {
>        return false;
>    }
>
>    // Recursively check the left and right subtrees with updated bounds
>    bool isLeftValid = isValidBST(node->left, minBound, node->val);
>    bool isRightValid = isValidBST(node->right, node->val, maxBound);
>
>    return isLeftValid && isRightValid;
>}
>```

>[!example]- Java
>```java
>public boolean isValidBST(TreeNode node, Long minBound, Long maxBound) {
>    // An empty tree is a valid BST
>    if (node == null) {
>        return true;
>    }
>
>    // Check if the current node's value is within its valid range
>    if ((minBound != null && node.val <= minBound) ||
>        (maxBound != null && node.val >= maxBound)) {
>        return false;
>    }
>
>    // Recursively check the left and right subtrees with updated bounds
>    boolean isLeftValid = isValidBST(node.left, minBound, (long)node.val);
>    boolean isRightValid = isValidBST(node.right, (long)node.val, maxBound);
>
>    return isLeftValid && isRightValid;
>}
>
>// Wrapper method for initial call
>public boolean isValidBST(TreeNode root) {
>    return isValidBST(root, null, null);
>}
>```

>[!example]- Python
>```python
>def is_valid_bst(node, min_bound=float('-inf'), max_bound=float('inf')):
>    # An empty tree is a valid BST
>    if not node:
>        return True
>
>    # Check if the current node's value is within its valid range
>    if not (min_bound < node.val < max_bound):
>        return False
>
>    # Recursively check the left and right subtrees with updated bounds
>    is_left_valid = is_valid_bst(node.left, min_bound, node.val)
>    is_right_valid = is_valid_bst(node.right, node.val, max_bound)
>
>    return is_left_valid and is_right_valid
>```

>[!example]- JavaScript
>```javascript
>function isValidBST(node, minBound = -Infinity, maxBound = Infinity) {
>    // An empty tree is a valid BST
>    if (!node) {
>        return true;
>    }
>
>    // Check if the current node's value is within its valid range
>    if (node.val <= minBound || node.val >= maxBound) {
>        return false;
>    }
>
>    // Recursively check the left and right subtrees with updated bounds
>    const isLeftValid = isValidBST(node.left, minBound, node.val);
>    const isRightValid = isValidBST(node.right, node.val, maxBound);
>
>    return isLeftValid && isRightValid;
>}
>```

This ensures that the BST property holds globally, not just locally.
