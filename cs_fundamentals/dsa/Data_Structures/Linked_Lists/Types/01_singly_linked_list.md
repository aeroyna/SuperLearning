## Singly Linked List

A singly linked list is the most common type of linked list. Each node contains data and a single pointer, `next`, that points to the next node in the sequence. This structure means you can only traverse the list in one direction: forward.

### Core Idea

- **Nodes**: Each `ListNode` holds a value (`val`) and a `next` pointer.
- **Head**: The entry point to the list.
- **Tail**: The last node in the list, whose `next` pointer is `null`.
- **Traversal**: You move from one node to the next by following the `next` pointers, i.e., `current = current.next`.

### Node Definition

>[!example]- C++
>```cpp
>struct ListNode {
>    int val;
>    ListNode* next;
>
>    ListNode(int x = 0, ListNode* n = nullptr) : val(x), next(n) {}
>};
>```

>[!example]- Java
>```java
>class ListNode {
>    int val;
>    ListNode next;
>
>    ListNode(int val) {
>        this.val = val;
>        this.next = null;
>    }
>
>    ListNode(int val, ListNode next) {
>        this.val = val;
>        this.next = next;
>    }
>}
>```

>[!example]- Python
>```python
>class ListNode:
>    def __init__(self, val=0, next=None):
>        self.val = val
>        self.next = next
>```

>[!example]- JavaScript
>```javascript
>class ListNode {
>    constructor(val = 0, next = null) {
>        this.val = val;
>        this.next = next;
>    }
>}
>```

### Key Operations

#### Insertion
To insert a new node, `node_to_add`, after a given `prev_node`:
1.  Point `node_to_add.next` to whatever `prev_node.next` was pointing to.
2.  Update `prev_node.next` to point to `node_to_add`.

This sequence is crucial to avoid losing the rest of the list.

>[!example]- C++
>```cpp
>void addNode(ListNode* prevNode, ListNode* nodeToAdd) {
>    // The new node must first point to the rest of the list
>    nodeToAdd->next = prevNode->next;
>    // Then the previous node can point to the new node
>    prevNode->next = nodeToAdd;
>}
>```

>[!example]- Java
>```java
>void addNode(ListNode prevNode, ListNode nodeToAdd) {
>    // The new node must first point to the rest of the list
>    nodeToAdd.next = prevNode.next;
>    // Then the previous node can point to the new node
>    prevNode.next = nodeToAdd;
>}
>```

>[!example]- Python
>```python
>def add_node(prev_node, node_to_add):
>    # The new node must first point to the rest of the list
>    node_to_add.next = prev_node.next
>    # Then the previous node can point to the new node
>    prev_node.next = node_to_add
>```

>[!example]- JavaScript
>```javascript
>function addNode(prevNode, nodeToAdd) {
>    // The new node must first point to the rest of the list
>    nodeToAdd.next = prevNode.next;
>    // Then the previous node can point to the new node
>    prevNode.next = nodeToAdd;
>}
>```

This is an O(1) operation, provided you have a reference to `prev_node`. Finding `prev_node` first would take O(n) time.

#### Deletion
To delete a node that comes after a given `prev_node`:
1.  Simply bypass the node to be deleted by pointing `prev_node.next` to the node *after* the one being deleted (`prev_node.next.next`).

>[!example]- C++
>```cpp
>void deleteNode(ListNode* prevNode) {
>    // Check if there is a node to delete
>    if (!prevNode || !prevNode->next) {
>        return;
>    }
>    // The 'next' pointer of the previous node skips over the target node
>    prevNode->next = prevNode->next->next;
>}
>```

>[!example]- Java
>```java
>void deleteNode(ListNode prevNode) {
>    // Check if there is a node to delete
>    if (prevNode == null || prevNode.next == null) {
>        return;
>    }
>    // The 'next' pointer of the previous node skips over the target node
>    prevNode.next = prevNode.next.next;
>}
>```

>[!example]- Python
>```python
>def delete_node(prev_node):
>    # Check if there is a node to delete
>    if not prev_node or not prev_node.next:
>        return
>    # The 'next' pointer of the previous node skips over the target node
>    prev_node.next = prev_node.next.next
>```

>[!example]- JavaScript
>```javascript
>function deleteNode(prevNode) {
>    // Check if there is a node to delete
>    if (!prevNode || !prevNode.next) {
>        return;
>    }
>    // The 'next' pointer of the previous node skips over the target node
>    prevNode.next = prevNode.next.next;
>}
>```

This is also an O(1) operation if you have `prev_node`. The need to have the *previous* node to perform efficient deletions is a key characteristic of singly linked lists.
