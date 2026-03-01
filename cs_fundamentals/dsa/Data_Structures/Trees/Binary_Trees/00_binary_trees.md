# Binary Trees

A binary tree is a hierarchical data structure where each node has at most two children: left and right.

## Overview

Trees are fundamental for representing hierarchical data and are heavily tested in interviews.

## Topics

- [7.1 Tree Traversals (DFS)](01_tree_traversals_dfs.md)
- [7.2 Tree Traversals (BFS)](02_tree_traversals_bfs.md)
- [7.3 Common Tree Problems](03_common_tree_problems.md)
- [7.4 Practice Problems](Practice_Problems/00_practice_problems.md)

## Tree Node Definition

```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right
```

## Terminology

- **Root**: Top node (no parent)
- **Leaf**: Node with no children
- **Height**: Longest path from root to leaf
- **Depth**: Distance from root to node
- **Subtree**: Tree formed by a node and all descendants

## DFS Traversal Orders

### Preorder (Root, Left, Right)

>[!example]- C++
>```cpp
>// Recursive
>void preorder(TreeNode* root, vector<int>& result) {
>    if (!root) return;
>    result.push_back(root->val);
>    preorder(root->left, result);
>    preorder(root->right, result);
>}
>
>// Iterative
>vector<int> preorderIterative(TreeNode* root) {
>    if (!root) return {};
>    vector<int> result;
>    stack<TreeNode*> s;
>    s.push(root);
>    while (!s.empty()) {
>        TreeNode* node = s.top();
>        s.pop();
>        result.push_back(node->val);
>        if (node->right) s.push(node->right);
>        if (node->left) s.push(node->left);
>    }
>    return result;
>}
>```

>[!example]- Java
>```java
>// Recursive
>public void preorder(TreeNode root, List<Integer> result) {
>    if (root == null) return;
>    result.add(root.val);
>    preorder(root.left, result);
>    preorder(root.right, result);
>}
>
>// Iterative
>public List<Integer> preorderIterative(TreeNode root) {
>    List<Integer> result = new ArrayList<>();
>    if (root == null) return result;
>    Stack<TreeNode> stack = new Stack<>();
>    stack.push(root);
>    while (!stack.isEmpty()) {
>        TreeNode node = stack.pop();
>        result.add(node.val);
>        if (node.right != null) stack.push(node.right);
>        if (node.left != null) stack.push(node.left);
>    }
>    return result;
>}
>```

>[!example]- Python
>```python
>def preorder(root):
>    if not root:
>        return []
>    return [root.val] + preorder(root.left) + preorder(root.right)
>
># Iterative
>def preorderIterative(root):
>    if not root:
>        return []
>    result, stack = [], [root]
>    while stack:
>        node = stack.pop()
>        result.append(node.val)
>        if node.right:
>            stack.append(node.right)
>        if node.left:
>            stack.append(node.left)
>    return result
>```

>[!example]- JavaScript
>```javascript
>// Recursive
>function preorder(root, result = []) {
>    if (!root) return result;
>    result.push(root.val);
>    preorder(root.left, result);
>    preorder(root.right, result);
>    return result;
>}
>
>// Iterative
>function preorderIterative(root) {
>    if (!root) return [];
>    const result = [];
>    const stack = [root];
>    while (stack.length > 0) {
>        const node = stack.pop();
>        result.push(node.val);
>        if (node.right) stack.push(node.right);
>        if (node.left) stack.push(node.left);
>    }
>    return result;
>}
>```

### Inorder (Left, Root, Right)

>[!example]- C++
>```cpp
>// Recursive
>void inorder(TreeNode* root, vector<int>& result) {
>    if (!root) return;
>    inorder(root->left, result);
>    result.push_back(root->val);
>    inorder(root->right, result);
>}
>
>// Iterative
>vector<int> inorderIterative(TreeNode* root) {
>    vector<int> result;
>    stack<TreeNode*> s;
>    TreeNode* current = root;
>    while (current || !s.empty()) {
>        while (current) {
>            s.push(current);
>            current = current->left;
>        }
>        current = s.top();
>        s.pop();
>        result.push_back(current->val);
>        current = current->right;
>    }
>    return result;
>}
>```

>[!example]- Java
>```java
>// Recursive
>public void inorder(TreeNode root, List<Integer> result) {
>    if (root == null) return;
>    inorder(root.left, result);
>    result.add(root.val);
>    inorder(root.right, result);
>}
>
>// Iterative
>public List<Integer> inorderIterative(TreeNode root) {
>    List<Integer> result = new ArrayList<>();
>    Stack<TreeNode> stack = new Stack<>();
>    TreeNode current = root;
>    while (current != null || !stack.isEmpty()) {
>        while (current != null) {
>            stack.push(current);
>            current = current.left;
>        }
>        current = stack.pop();
>        result.add(current.val);
>        current = current.right;
>    }
>    return result;
>}
>```

>[!example]- Python
>```python
>def inorder(root):
>    if not root:
>        return []
>    return inorder(root.left) + [root.val] + inorder(root.right)
>
># Iterative
>def inorderIterative(root):
>    result, stack = [], []
>    current = root
>    while current or stack:
>        while current:
>            stack.append(current)
>            current = current.left
>        current = stack.pop()
>        result.append(current.val)
>        current = current.right
>    return result
>```

>[!example]- JavaScript
>```javascript
>// Recursive
>function inorder(root, result = []) {
>    if (!root) return result;
>    inorder(root.left, result);
>    result.push(root.val);
>    inorder(root.right, result);
>    return result;
>}
>
>// Iterative
>function inorderIterative(root) {
>    const result = [];
>    const stack = [];
>    let current = root;
>    while (current || stack.length > 0) {
>        while (current) {
>            stack.push(current);
>            current = current.left;
>        }
>        current = stack.pop();
>        result.push(current.val);
>        current = current.right;
>    }
>    return result;
>}
>```

### Postorder (Left, Right, Root)

```python
def postorder(root):
    if not root:
        return []
    return postorder(root.left) + postorder(root.right) + [root.val]
```

## BFS (Level Order)

```python
from collections import deque

def levelOrder(root):
    if not root:
        return []

    result = []
    queue = deque([root])

    while queue:
        level = []
        for _ in range(len(queue)):
            node = queue.popleft()
            level.append(node.val)
            if node.left:
                queue.append(node.left)
            if node.right:
                queue.append(node.right)
        result.append(level)

    return result
```

## Common DFS Pattern

Most tree problems use this recursive pattern:

```python
def dfs(node):
    # Base case
    if not node:
        return base_value

    # Recursive calls
    left_result = dfs(node.left)
    right_result = dfs(node.right)

    # Combine results
    return combine(node.val, left_result, right_result)
```

### Example: Maximum Depth

```python
def maxDepth(root):
    if not root:
        return 0
    return 1 + max(maxDepth(root.left), maxDepth(root.right))
```

### Example: Path Sum

```python
def hasPathSum(root, targetSum):
    if not root:
        return False
    if not root.left and not root.right:
        return root.val == targetSum
    return (hasPathSum(root.left, targetSum - root.val) or
            hasPathSum(root.right, targetSum - root.val))
```

## Key Interview Problems

| Problem | Pattern | Difficulty | LeetCode Link |
| --------- | --------- | ------------ | --- |
| Maximum Depth | DFS | Easy | [Link](https://leetcode.com/problems/maximum-depth/) |
| Same Tree | DFS Compare | Easy | [Link](https://leetcode.com/problems/same-tree/) |
| Invert Binary Tree | DFS Modify | Easy | [Link](https://leetcode.com/problems/invert-binary-tree/) |
| Path Sum | DFS with Target | Easy | [Link](https://leetcode.com/problems/path-sum/) |
| Level Order Traversal | BFS | Medium | [Link](https://leetcode.com/problems/level-order-traversal/) |
| Validate BST | Inorder/Range | Medium | [Link](https://leetcode.com/problems/validate-bst/) |
| Lowest Common Ancestor | DFS | Medium | [Link](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree/) |
| Serialize/Deserialize | BFS/DFS | Hard | [Link](https://leetcode.com/problems/serializedeserialize/) |
