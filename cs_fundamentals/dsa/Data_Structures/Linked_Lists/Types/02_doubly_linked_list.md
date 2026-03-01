## Doubly Linked List

A doubly linked list enhances the singly linked list by adding a second pointer, `prev`, to each node. This allows for bidirectional traversal, meaning you can move both forwards and backwards through the list.

### Core Idea

- **Nodes**: Each `ListNode` contains a value (`val`), a `next` pointer, and a `prev` pointer.
- **`next` pointer**: Points to the subsequent node in the list.
- **`prev` pointer**: Points to the preceding node in the list. The `prev` pointer of the head node is `null`.

This two-way structure makes some operations more efficient than in a singly linked list.

### Node Definition

>[!example]- C++
>```cpp
>struct ListNode {
>    int val;
>    ListNode* next;
>    ListNode* prev;
>
>    ListNode(int x = 0, ListNode* n = nullptr, ListNode* p = nullptr)
>        : val(x), next(n), prev(p) {}
>};
>```

>[!example]- Java
>```java
>class ListNode {
>    int val;
>    ListNode next;
>    ListNode prev;
>
>    ListNode(int val) {
>        this.val = val;
>        this.next = null;
>        this.prev = null;
>    }
>
>    ListNode(int val, ListNode next, ListNode prev) {
>        this.val = val;
>        this.next = next;
>        this.prev = prev;
>    }
>}
>```

>[!example]- Python
>```python
>class ListNode:
>    def __init__(self, val=0, next=None, prev=None):
>        self.val = val
>        self.next = next
>        self.prev = prev
>```

>[!example]- JavaScript
>```javascript
>class ListNode {
>    constructor(val = 0, next = null, prev = null) {
>        this.val = val;
>        this.next = next;
>        this.prev = prev;
>    }
>}
>```

### Key Operations

The primary advantage of a doubly linked list is the ability to perform deletions in O(1) time even if you only have a reference to the node being deleted, as you can access its previous node via the `prev` pointer.

#### Insertion
To insert `node_to_add` before a given `node`:
1.  Connect `node_to_add` to its surrounding nodes (`node` and `node.prev`).
2.  Update the surrounding nodes to point to `node_to_add`.

>[!example]- C++
>```cpp
>void addNodeBefore(ListNode* node, ListNode* nodeToAdd) {
>    ListNode* prevNode = node->prev;
>
>    // Link the new node to its neighbors
>    nodeToAdd->next = node;
>    nodeToAdd->prev = prevNode;
>
>    // Update the neighbors' pointers to link to the new node
>    node->prev = nodeToAdd;
>    if (prevNode) {
>        prevNode->next = nodeToAdd;
>    }
>}
>```

>[!example]- Java
>```java
>void addNodeBefore(ListNode node, ListNode nodeToAdd) {
>    ListNode prevNode = node.prev;
>
>    // Link the new node to its neighbors
>    nodeToAdd.next = node;
>    nodeToAdd.prev = prevNode;
>
>    // Update the neighbors' pointers to link to the new node
>    node.prev = nodeToAdd;
>    if (prevNode != null) {
>        prevNode.next = nodeToAdd;
>    }
>}
>```

>[!example]- Python
>```python
>def add_node_before(node, node_to_add):
>    prev_node = node.prev
>
>    # Link the new node to its neighbors
>    node_to_add.next = node
>    node_to_add.prev = prev_node
>
>    # Update the neighbors' pointers to link to the new node
>    node.prev = node_to_add
>    if prev_node:
>        prev_node.next = node_to_add
>```

>[!example]- JavaScript
>```javascript
>function addNodeBefore(node, nodeToAdd) {
>    const prevNode = node.prev;
>
>    // Link the new node to its neighbors
>    nodeToAdd.next = node;
>    nodeToAdd.prev = prevNode;
>
>    // Update the neighbors' pointers to link to the new node
>    node.prev = nodeToAdd;
>    if (prevNode) {
>        prevNode.next = nodeToAdd;
>    }
>}
>```

#### Deletion
To delete a `node`:
1.  Identify its `prev` and `next` nodes.
2.  Connect the `prev` and `next` nodes directly to each other, bypassing the current `node`.

>[!example]- C++
>```cpp
>void deleteNode(ListNode* node) {
>    ListNode* prevNode = node->prev;
>    ListNode* nextNode = node->next;
>
>    // Bypass the current node
>    if (prevNode) {
>        prevNode->next = nextNode;
>    }
>    if (nextNode) {
>        nextNode->prev = prevNode;
>    }
>}
>```

>[!example]- Java
>```java
>void deleteNode(ListNode node) {
>    ListNode prevNode = node.prev;
>    ListNode nextNode = node.next;
>
>    // Bypass the current node
>    if (prevNode != null) {
>        prevNode.next = nextNode;
>    }
>    if (nextNode != null) {
>        nextNode.prev = prevNode;
>    }
>}
>```

>[!example]- Python
>```python
>def delete_node(node):
>    prev_node = node.prev
>    next_node = node.next
>
>    # Bypass the current node
>    if prev_node:
>        prev_node.next = next_node
>    if next_node:
>        next_node.prev = prev_node
>```

>[!example]- JavaScript
>```javascript
>function deleteNode(node) {
>    const prevNode = node.prev;
>    const nextNode = node.next;
>
>    // Bypass the current node
>    if (prevNode) {
>        prevNode.next = nextNode;
>    }
>    if (nextNode) {
>        nextNode.prev = prevNode;
>    }
>}
>```

This O(1) deletion is a significant advantage over singly linked lists, where you would first need to spend O(n) time finding the previous node.

### Use Cases
- Implementing stacks and queues (deques).
- Building the foundation for more complex data structures like LRU Caches, where you need to move nodes to the front or back of a list efficiently.
