## Tree Traversals: Depth-First Search (DFS)

Depth-First Search (DFS) is a fundamental way to traverse a tree. It explores as far as possible down one branch before backtracking. For any given node, it will explore its entire left subtree before it even begins to explore its right subtree. This "depth-first" approach is naturally implemented using recursion.

There are three primary DFS traversal orders. The only difference between them is the order in which the current node is processed relative to its left and right children.

---

### 1. Pre-order Traversal (Node → Left → Right)

In a pre-order traversal, the current node is processed *first*, followed by a recursive traversal of the left subtree, and then the right subtree.

- **Use Case**: Creating a copy of a tree. You can create the new root node, then recursively call on the children to create their subtrees.

>[!example]- C++
>```cpp
>void preorderTraversal(TreeNode* node, vector<int>& result) {
>    // Recursively traverses a tree in Pre-order
>    if (!node) {
>        return;
>    }
>
>    result.push_back(node->val);           // 1. Process the Node
>    preorderTraversal(node->left, result);  // 2. Recurse on Left
>    preorderTraversal(node->right, result); // 3. Recurse on Right
>}
>```

>[!example]- Java
>```java
>public void preorderTraversal(TreeNode node, List<Integer> result) {
>    // Recursively traverses a tree in Pre-order
>    if (node == null) {
>        return;
>    }
>
>    result.add(node.val);                      // 1. Process the Node
>    preorderTraversal(node.left, result);      // 2. Recurse on Left
>    preorderTraversal(node.right, result);     // 3. Recurse on Right
>}
>```

>[!example]- Python
>```python
>def preorder_traversal(node, result_list):
>    """Recursively traverses a tree in Pre-order."""
>    if not node:
>        return
>
>    result_list.append(node.val)      # 1. Process the Node
>    preorder_traversal(node.left, result_list)   # 2. Recurse on Left
>    preorder_traversal(node.right, result_list)  # 3. Recurse on Right
>```

>[!example]- JavaScript
>```javascript
>function preorderTraversal(node, result) {
>    // Recursively traverses a tree in Pre-order
>    if (!node) {
>        return;
>    }
>
>    result.push(node.val);                  // 1. Process the Node
>    preorderTraversal(node.left, result);   // 2. Recurse on Left
>    preorderTraversal(node.right, result);  // 3. Recurse on Right
>}
>```

---

### 2. In-order Traversal (Left → Node → Right)

In an in-order traversal, the left subtree is fully traversed *first*, then the current node is processed, and finally, the right subtree is traversed.

- **Use Case**: When applied to a Binary Search Tree (BST), an in-order traversal visits the nodes in their natural, sorted order (ascending). This is a critical property of BSTs.

>[!example]- C++
>```cpp
>void inorderTraversal(TreeNode* node, vector<int>& result) {
>    // Recursively traverses a tree in In-order
>    if (!node) {
>        return;
>    }
>
>    inorderTraversal(node->left, result);  // 1. Recurse on Left
>    result.push_back(node->val);           // 2. Process the Node
>    inorderTraversal(node->right, result); // 3. Recurse on Right
>}
>```

>[!example]- Java
>```java
>public void inorderTraversal(TreeNode node, List<Integer> result) {
>    // Recursively traverses a tree in In-order
>    if (node == null) {
>        return;
>    }
>
>    inorderTraversal(node.left, result);       // 1. Recurse on Left
>    result.add(node.val);                      // 2. Process the Node
>    inorderTraversal(node.right, result);      // 3. Recurse on Right
>}
>```

>[!example]- Python
>```python
>def inorder_traversal(node, result_list):
>    """Recursively traverses a tree in In-order."""
>    if not node:
>        return
>
>    inorder_traversal(node.left, result_list)   # 1. Recurse on Left
>    result_list.append(node.val)      # 2. Process the Node
>    inorder_traversal(node.right, result_list)  # 3. Recurse on Right
>```

>[!example]- JavaScript
>```javascript
>function inorderTraversal(node, result) {
>    // Recursively traverses a tree in In-order
>    if (!node) {
>        return;
>    }
>
>    inorderTraversal(node.left, result);    // 1. Recurse on Left
>    result.push(node.val);                  // 2. Process the Node
>    inorderTraversal(node.right, result);   // 3. Recurse on Right
>}
>```

---

### 3. Post-order Traversal (Left → Right → Node)

In a post-order traversal, both the left and right subtrees are fully traversed *before* the current node is processed.

- **Use Case**: This is useful when the parent node's processing depends on the results from its children. A common example is deleting nodes in a tree (you must delete the children before you can delete the parent) or calculating the height of a tree.

>[!example]- C++
>```cpp
>void postorderTraversal(TreeNode* node, vector<int>& result) {
>    // Recursively traverses a tree in Post-order
>    if (!node) {
>        return;
>    }
>
>    postorderTraversal(node->left, result);  // 1. Recurse on Left
>    postorderTraversal(node->right, result); // 2. Recurse on Right
>    result.push_back(node->val);             // 3. Process the Node
>}
>```

>[!example]- Java
>```java
>public void postorderTraversal(TreeNode node, List<Integer> result) {
>    // Recursively traverses a tree in Post-order
>    if (node == null) {
>        return;
>    }
>
>    postorderTraversal(node.left, result);     // 1. Recurse on Left
>    postorderTraversal(node.right, result);    // 2. Recurse on Right
>    result.add(node.val);                      // 3. Process the Node
>}
>```

>[!example]- Python
>```python
>def postorder_traversal(node, result_list):
>    """Recursively traverses a tree in Post-order."""
>    if not node:
>        return
>
>    postorder_traversal(node.left, result_list)   # 1. Recurse on Left
>    postorder_traversal(node.right, result_list)  # 2. Recurse on Right
>    result_list.append(node.val)      # 3. Process the Node
>```

>[!example]- JavaScript
>```javascript
>function postorderTraversal(node, result) {
>    // Recursively traverses a tree in Post-order
>    if (!node) {
>        return;
>    }
>
>    postorderTraversal(node.left, result);   // 1. Recurse on Left
>    postorderTraversal(node.right, result);  // 2. Recurse on Right
>    result.push(node.val);                   // 3. Process the Node
>}
>```
