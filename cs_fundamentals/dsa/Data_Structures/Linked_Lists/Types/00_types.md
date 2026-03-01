# Linked List Types

Linked lists come in several variations, each offering different trade-offs between memory overhead, traversal capabilities, and operational complexity. Understanding these types is essential for selecting the right implementation.

## Overview

The core variations differ in pointer structure:
- **Singly Linked**: One pointer (next), forward traversal only
- **Doubly Linked**: Two pointers (next, prev), bidirectional traversal
- **Circular**: Tail connects back to head, enabling continuous iteration

## Topics

- [4.1.1 Singly Linked List](01_singly_linked_list.md)
- [4.1.2 Doubly Linked List](02_doubly_linked_list.md)

## Memory Layout Comparison

### Singly Linked List

```
Node:  [data | next] → [data | next] → [data | next] → null
Size:  8 bytes data + 8 bytes pointer = 16 bytes per node (64-bit system)
```

### Doubly Linked List

```
Node:  null ← [prev | data | next] ↔ [prev | data | next] ↔ [prev | data | next] → null
Size:  8 bytes data + 8 bytes prev + 8 bytes next = 24 bytes per node
```

**Memory overhead insight**: Doubly linked lists use 50% more memory per node than singly linked lists. For a million-node list, that's an extra 8MB just for backward pointers.

## Complexity Comparison

| Operation | Singly | Doubly | Notes |
|-----------|--------|--------|-------|
| Access by index | O(n) | O(n) | Must traverse |
| Insert at head | O(1) | O(1) | Update head pointer |
| Insert at tail | O(n) / O(1)* | O(1) | *With tail pointer |
| Delete at head | O(1) | O(1) | Update head pointer |
| Delete given node | O(n) | O(1) | Singly needs previous |
| Reverse traversal | O(n²) / impossible | O(n) | Doubly has prev pointers |

## When to Use Each Type

### Singly Linked List

**Use when**:
- Memory is constrained
- Only forward traversal needed
- Implementing stacks (LIFO)
- Simple insertion/deletion at head

>[!example]- C++
>```cpp
>struct SinglyNode {
>    int val;
>    SinglyNode *next;
>    SinglyNode(int x) : val(x), next(nullptr) {}
>};
>```

>[!example]- Java
>```java
>class SinglyNode {
>    int val;
>    SinglyNode next;
>    SinglyNode(int val) { this.val = val; }
>}
>```

>[!example]- Python
>```python
>class SinglyNode:
>    def __init__(self, val=0, next=None):
>        self.val = val
>        self.next = next
>```

>[!example]- JavaScript
>```javascript
>class SinglyNode {
>    constructor(val = 0, next = null) {
>        this.val = val;
>        this.next = next;
>    }
>}
>```

### Doubly Linked List

**Use when**:
- Need backward traversal
- Frequent deletion of arbitrary nodes (given reference)
- Implementing LRU cache (need O(1) move-to-front)
- Implementing deque operations

>[!example]- C++
>```cpp
>struct DoublyNode {
>    int val;
>    DoublyNode *prev;
>    DoublyNode *next;
>    DoublyNode(int x) : val(x), prev(nullptr), next(nullptr) {}
>};
>```

>[!example]- Java
>```java
>class DoublyNode {
>    int val;
>    DoublyNode prev;
>    DoublyNode next;
>    DoublyNode(int val) { this.val = val; }
>}
>```

>[!example]- Python
>```python
>class DoublyNode:
>    def __init__(self, val=0, prev=None, next=None):
>        self.val = val
>        self.prev = prev
>        self.next = next
>```

>[!example]- JavaScript
>```javascript
>class DoublyNode {
>    constructor(val = 0, prev = null, next = null) {
>        this.val = val;
>        this.prev = prev;
>        this.next = next;
>    }
>}
>```

### Circular Linked List

**Use when**:
- Round-robin scheduling
- Circular buffers
- Josephus problem
- Continuous cycling through elements

## Architectural Decision: Sentinel Nodes

Using dummy head (and tail) nodes simplifies edge cases:

>[!example]- C++
>```cpp
>class DoublyLinkedList {
>    DoublyNode *head, *tail;
>public:
>    DoublyLinkedList() {
>        head = new DoublyNode(0);
>        tail = new DoublyNode(0);
>        head->next = tail;
>        tail->prev = head;
>    }
>    void insertAfter(DoublyNode* node, DoublyNode* newNode) {
>        newNode->prev = node;
>        newNode->next = node->next;
>        node->next->prev = newNode;
>        node->next = newNode;
>    }
>};
>```

>[!example]- Java
>```java
>class DoublyLinkedList {
>    DoublyNode head, tail;
>    public DoublyLinkedList() {
>        head = new DoublyNode(0);
>        tail = new DoublyNode(0);
>        head.next = tail;
>        tail.prev = head;
>    }
>    public void insertAfter(DoublyNode node, DoublyNode newNode) {
>        newNode.prev = node;
>        newNode.next = node.next;
>        node.next.prev = newNode;
>        node.next = newNode;
>    }
>}
>```

>[!example]- Python
>```python
>class DoublyLinkedList:
>    def __init__(self):
>        self.head = DoublyNode()  # Sentinel
>        self.tail = DoublyNode()  # Sentinel
>        self.head.next = self.tail
>        self.tail.prev = self.head
>
>    def insert_after(self, node, new_node):
>        # No null checks needed
>        new_node.prev = node
>        new_node.next = node.next
>        node.next.prev = new_node
>        node.next = new_node
>```

>[!example]- JavaScript
>```javascript
>class DoublyLinkedList {
>    constructor() {
>        this.head = new DoublyNode(0);
>        this.tail = new DoublyNode(0);
>        this.head.next = this.tail;
>        this.tail.prev = this.head;
>    }
>    insertAfter(node, newNode) {
>        newNode.prev = node;
>        newNode.next = node.next;
>        node.next.prev = newNode;
>        node.next = newNode;
>    }
>}
>```

**Why sentinels help**: Without them, every operation needs `if head is None` and `if node.next is None` checks. Sentinels guarantee `prev` and `next` always exist.

## Common Pitfalls

1. **Losing the head reference**: Always maintain a reference to the head
2. **Not updating all pointers**: In doubly linked lists, forgetting to update `prev`
3. **Memory leaks**: In languages without GC, not freeing deleted nodes
4. **Circular reference bugs**: Creating unintended cycles that cause infinite loops
