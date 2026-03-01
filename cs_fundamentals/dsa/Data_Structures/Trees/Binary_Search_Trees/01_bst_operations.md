## Operations on Binary Search Trees

The ordering property of a BST makes its core operations (search, insert, and delete) very efficient. These operations leverage a binary search-like approach, leading to an average time complexity of O(log n).

### Search
Searching for a value in a BST is a straightforward traversal.
1.  Start at the `root`.
2.  Compare the target value with the current node's value.
3.  - If the target matches, the value is found.
    - If the target is **less** than the current node's value, move to the **left** child.
    - If the target is **greater** than the current node's value, move to the **right** child.
4.  Repeat until the value is found or you reach a `null` node (meaning the value is not in the tree).

>[!example]- C++
>```cpp
>TreeNode* searchBST(TreeNode* root, int val) {
>    TreeNode* curr = root;
>    while (curr) {
>        if (val < curr->val) {
>            curr = curr->left;
>        } else if (val > curr->val) {
>            curr = curr->right;
>        } else {
>            return curr; // Value found
>        }
>    }
>    return nullptr; // Value not in tree
>}
>```

>[!example]- Java
>```java
>public TreeNode searchBST(TreeNode root, int val) {
>    TreeNode curr = root;
>    while (curr != null) {
>        if (val < curr.val) {
>            curr = curr.left;
>        } else if (val > curr.val) {
>            curr = curr.right;
>        } else {
>            return curr; // Value found
>        }
>    }
>    return null; // Value not in tree
>}
>```

>[!example]- Python
>```python
>def search_bst(root, val):
>    curr = root
>    while curr:
>        if val < curr.val:
>            curr = curr.left
>        elif val > curr.val:
>            curr = curr.right
>        else:
>            return curr # Value found
>    return None # Value not in tree
>```

>[!example]- JavaScript
>```javascript
>function searchBST(root, val) {
>    let curr = root;
>    while (curr) {
>        if (val < curr.val) {
>            curr = curr.left;
>        } else if (val > curr.val) {
>            curr = curr.right;
>        } else {
>            return curr; // Value found
>        }
>    }
>    return null; // Value not in tree
>}
>```

### Insertion
Insertion follows the same logic as searching to find the correct position for the new node.
1.  Traverse the tree as if you were searching for the value to be inserted.
2.  When you reach a `null` child pointer (left or right), that is the location where the new node should be inserted to maintain the BST property.

>[!example]- C++
>```cpp
>TreeNode* insertIntoBST(TreeNode* root, int val) {
>    if (!root) {
>        return new TreeNode(val); // If tree is empty, new node is the root
>    }
>
>    TreeNode* curr = root;
>    while (true) {
>        if (val < curr->val) {
>            if (!curr->left) {
>                curr->left = new TreeNode(val);
>                return root;
>            }
>            curr = curr->left;
>        } else { // val > curr->val (duplicates are not allowed)
>            if (!curr->right) {
>                curr->right = new TreeNode(val);
>                return root;
>            }
>            curr = curr->right;
>        }
>    }
>}
>```

>[!example]- Java
>```java
>public TreeNode insertIntoBST(TreeNode root, int val) {
>    if (root == null) {
>        return new TreeNode(val); // If tree is empty, new node is the root
>    }
>
>    TreeNode curr = root;
>    while (true) {
>        if (val < curr.val) {
>            if (curr.left == null) {
>                curr.left = new TreeNode(val);
>                return root;
>            }
>            curr = curr.left;
>        } else { // val > curr.val (duplicates are not allowed)
>            if (curr.right == null) {
>                curr.right = new TreeNode(val);
>                return root;
>            }
>            curr = curr.right;
>        }
>    }
>}
>```

>[!example]- Python
>```python
>def insert_into_bst(root, val):
>    if not root:
>        return TreeNode(val) # If tree is empty, new node is the root
>
>    curr = root
>    while True:
>        if val < curr.val:
>            if not curr.left:
>                curr.left = TreeNode(val)
>                return root
>            curr = curr.left
>        else: # val > curr.val (duplicates are not allowed)
>            if not curr.right:
>                curr.right = TreeNode(val)
>                return root
>            curr = curr.right
>```

>[!example]- JavaScript
>```javascript
>function insertIntoBST(root, val) {
>    if (!root) {
>        return new TreeNode(val); // If tree is empty, new node is the root
>    }
>
>    let curr = root;
>    while (true) {
>        if (val < curr.val) {
>            if (!curr.left) {
>                curr.left = new TreeNode(val);
>                return root;
>            }
>            curr = curr.left;
>        } else { // val > curr.val (duplicates are not allowed)
>            if (!curr.right) {
>                curr.right = new TreeNode(val);
>                return root;
>            }
>            curr = curr.right;
>        }
>    }
>}
>```

### Deletion
Deletion is the most complex operation because the BST property must be maintained after the node is removed. There are three cases for the node to be deleted:

1.  **Node is a leaf (no children)**: Simply remove the node by setting its parent's corresponding child pointer to `null`.
2.  **Node has one child**: Bypass the node by linking its parent directly to its child.
3.  **Node has two children**: This is the tricky case. To maintain the BST property, you must replace the deleted node with either:
    -   The **in-order successor** (the smallest node in its right subtree).
    -   The **in-order predecessor** (the largest node in its left subtree).

    The chosen successor/predecessor is moved into the position of the deleted node, and then the successor/predecessor is deleted from its original position (which is an easier deletion case).

Due to its complexity, implementing BST deletion from scratch is a less common interview question than search, insertion, or validation, but understanding the logic is important.
