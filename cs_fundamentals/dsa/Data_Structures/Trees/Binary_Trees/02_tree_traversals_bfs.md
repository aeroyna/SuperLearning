## Tree Traversals: Breadth-First Search (BFS)

Breadth-First Search (BFS), also known as Level-Order Traversal, is the other major strategy for visiting nodes in a tree. Instead of going deep like DFS, BFS explores the tree layer by layer. It visits all nodes at a certain depth before moving on to the next deeper level.

### Core Idea
BFS uses a **queue** to manage the order of nodes to visit. The process works as follows:
1.  Start by adding the `root` node to the queue.
2.  While the queue is not empty:
    a. Dequeue a node.
    b. Process the dequeued node (e.g., add its value to a result list).
    c. Enqueue its children (typically left child first, then right child).

This FIFO (First-In, First-Out) process naturally results in a level-by-level traversal.

### When to Use BFS
BFS is the ideal choice for any problem involving the **levels** of a tree or finding the **shortest path** from the root to a target node. Common problem statements include:
- Find the minimum depth of a tree.
- Find the largest value on each level.
- Get the "right side view" of a tree (the last node on each level).
- Any problem asking for the "shortest," "fewest," or "minimum" number of steps in a tree context.

### Implementation
A standard BFS implementation processes one level at a time. This is done by first getting the number of nodes currently in the queue (the size of the current level) and then running a loop that many times.

>[!example]- C++
>```cpp
>vector<vector<int>> levelOrderTraversal(TreeNode* root) {
>    // Traverses a tree in Level-order (BFS) and returns a list of levels
>    if (!root) {
>        return {};
>    }
>
>    queue<TreeNode*> q;
>    q.push(root);
>    vector<vector<int>> result;
>
>    while (!q.empty()) {
>        // Number of nodes at the current level
>        int levelSize = q.size();
>        vector<int> currentLevel;
>
>        // Process all nodes for the current level
>        for (int i = 0; i < levelSize; i++) {
>            TreeNode* node = q.front();
>            q.pop();
>            currentLevel.push_back(node->val);
>
>            // Add children to the queue for the next level's processing
>            if (node->left) {
>                q.push(node->left);
>            }
>            if (node->right) {
>                q.push(node->right);
>            }
>        }
>
>        result.push_back(currentLevel);
>    }
>
>    return result;
>}
>```

>[!example]- Java
>```java
>public List<List<Integer>> levelOrderTraversal(TreeNode root) {
>    // Traverses a tree in Level-order (BFS) and returns a list of levels
>    if (root == null) {
>        return new ArrayList<>();
>    }
>
>    Queue<TreeNode> queue = new LinkedList<>();
>    queue.offer(root);
>    List<List<Integer>> result = new ArrayList<>();
>
>    while (!queue.isEmpty()) {
>        // Number of nodes at the current level
>        int levelSize = queue.size();
>        List<Integer> currentLevel = new ArrayList<>();
>
>        // Process all nodes for the current level
>        for (int i = 0; i < levelSize; i++) {
>            TreeNode node = queue.poll();
>            currentLevel.add(node.val);
>
>            // Add children to the queue for the next level's processing
>            if (node.left != null) {
>                queue.offer(node.left);
>            }
>            if (node.right != null) {
>                queue.offer(node.right);
>            }
>        }
>
>        result.add(currentLevel);
>    }
>
>    return result;
>}
>```

>[!example]- Python
>```python
>from collections import deque
>
>def level_order_traversal(root):
>    """Traverses a tree in Level-order (BFS) and returns a list of levels."""
>    if not root:
>        return []
>
>    queue = deque([root])
>    result = []
>
>    while queue:
>        # Number of nodes at the current level
>        level_size = len(queue)
>        current_level = []
>
>        # Process all nodes for the current level
>        for _ in range(level_size):
>            node = queue.popleft()
>            current_level.append(node.val)
>
>            # Add children to the queue for the next level's processing
>            if node.left:
>                queue.append(node.left)
>            if node.right:
>                queue.append(node.right)
>
>        result.append(current_level)
>
>    return result
>```

>[!example]- JavaScript
>```javascript
>function levelOrderTraversal(root) {
>    // Traverses a tree in Level-order (BFS) and returns a list of levels
>    if (!root) {
>        return [];
>    }
>
>    const queue = [root];
>    const result = [];
>
>    while (queue.length > 0) {
>        // Number of nodes at the current level
>        const levelSize = queue.length;
>        const currentLevel = [];
>
>        // Process all nodes for the current level
>        for (let i = 0; i < levelSize; i++) {
>            const node = queue.shift();
>            currentLevel.push(node.val);
>
>            // Add children to the queue for the next level's processing
>            if (node.left) {
>                queue.push(node.left);
>            }
>            if (node.right) {
>                queue.push(node.right);
>            }
>        }
>
>        result.push(currentLevel);
>    }
>
>    return result;
>}
>```

This level-by-level processing is a powerful pattern for solving many tree problems efficiently.
