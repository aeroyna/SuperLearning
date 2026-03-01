# Reversing Linked Lists

Reversing a linked list is a fundamental operation that appears in many interview problems.

## Iterative Reversal

### Basic Template

>[!example]- C++
>```cpp
>ListNode* reverseList(ListNode* head) {
>    ListNode* prev = nullptr;
>    ListNode* current = head;
>
>    while (current) {
>        ListNode* nextNode = current->next;  // Save next
>        current->next = prev;                // Reverse link
>        prev = current;                      // Move prev forward
>        current = nextNode;                  // Move current forward
>    }
>
>    return prev;  // New head
>}
>```

>[!example]- Java
>```java
>ListNode reverseList(ListNode head) {
>    ListNode prev = null;
>    ListNode current = head;
>
>    while (current != null) {
>        ListNode nextNode = current.next;  // Save next
>        current.next = prev;               // Reverse link
>        prev = current;                    // Move prev forward
>        current = nextNode;                // Move current forward
>    }
>
>    return prev;  // New head
>}
>```

>[!example]- Python
>```python
>def reverseList(head):
>    prev = None
>    current = head
>
>    while current:
>        next_node = current.next  # Save next
>        current.next = prev       # Reverse link
>        prev = current           # Move prev forward
>        current = next_node      # Move current forward
>
>    return prev  # New head
>```

>[!example]- JavaScript
>```javascript
>function reverseList(head) {
>    let prev = null;
>    let current = head;
>
>    while (current) {
>        const nextNode = current.next;  // Save next
>        current.next = prev;            // Reverse link
>        prev = current;                 // Move prev forward
>        current = nextNode;             // Move current forward
>    }
>
>    return prev;  // New head
>}
>```

### Step-by-Step Visualization

```
Original: 1 -> 2 -> 3 -> None

Step 1: prev=None, curr=1
        None <- 1    2 -> 3 -> None
        prev=1, curr=2

Step 2: prev=1, curr=2
        None <- 1 <- 2    3 -> None
        prev=2, curr=3

Step 3: prev=2, curr=3
        None <- 1 <- 2 <- 3    None
        prev=3, curr=None

Result: 3 -> 2 -> 1 -> None
```

## Recursive Reversal

### Basic Template

>[!example]- C++
>```cpp
>ListNode* reverseListRecursive(ListNode* head) {
>    // Base case
>    if (!head || !head->next) {
>        return head;
>    }
>
>    // Recurse to end
>    ListNode* newHead = reverseListRecursive(head->next);
>
>    // Reverse the link
>    head->next->next = head;
>    head->next = nullptr;
>
>    return newHead;
>}
>```

>[!example]- Java
>```java
>ListNode reverseListRecursive(ListNode head) {
>    // Base case
>    if (head == null || head.next == null) {
>        return head;
>    }
>
>    // Recurse to end
>    ListNode newHead = reverseListRecursive(head.next);
>
>    // Reverse the link
>    head.next.next = head;
>    head.next = null;
>
>    return newHead;
>}
>```

>[!example]- Python
>```python
>def reverseListRecursive(head):
>    # Base case
>    if not head or not head.next:
>        return head
>
>    # Recurse to end
>    new_head = reverseListRecursive(head.next)
>
>    # Reverse the link
>    head.next.next = head
>    head.next = None
>
>    return new_head
>```

>[!example]- JavaScript
>```javascript
>function reverseListRecursive(head) {
>    // Base case
>    if (!head || !head.next) {
>        return head;
>    }
>
>    // Recurse to end
>    const newHead = reverseListRecursive(head.next);
>
>    // Reverse the link
>    head.next.next = head;
>    head.next = null;
>
>    return newHead;
>}
>```

### How Recursion Works

```
reverseList(1)
    reverseList(2)
        reverseList(3) -> returns 3 (base case)
        3.next = 2, 2.next = None
        return 3
    2.next = 1, 1.next = None
    return 3
return 3

Result: 3 -> 2 -> 1 -> None
```

## Reverse Portion of List

Reverse nodes from position `left` to `right`.

>[!example]- C++
>```cpp
>ListNode* reverseBetween(ListNode* head, int left, int right) {
>    if (!head || left == right) {
>        return head;
>    }
>
>    ListNode* dummy = new ListNode(0, head);
>    ListNode* prev = dummy;
>
>    // Move to node before left
>    for (int i = 0; i < left - 1; i++) {
>        prev = prev->next;
>    }
>
>    // Reverse from left to right
>    ListNode* current = prev->next;
>    for (int i = 0; i < right - left; i++) {
>        ListNode* nextNode = current->next;
>        current->next = nextNode->next;
>        nextNode->next = prev->next;
>        prev->next = nextNode;
>    }
>
>    return dummy->next;
>}
>```

>[!example]- Java
>```java
>ListNode reverseBetween(ListNode head, int left, int right) {
>    if (head == null || left == right) {
>        return head;
>    }
>
>    ListNode dummy = new ListNode(0, head);
>    ListNode prev = dummy;
>
>    // Move to node before left
>    for (int i = 0; i < left - 1; i++) {
>        prev = prev.next;
>    }
>
>    // Reverse from left to right
>    ListNode current = prev.next;
>    for (int i = 0; i < right - left; i++) {
>        ListNode nextNode = current.next;
>        current.next = nextNode.next;
>        nextNode.next = prev.next;
>        prev.next = nextNode;
>    }
>
>    return dummy.next;
>}
>```

>[!example]- Python
>```python
>def reverseBetween(head, left, right):
>    if not head or left == right:
>        return head
>
>    dummy = ListNode(0, head)
>    prev = dummy
>
>    # Move to node before left
>    for _ in range(left - 1):
>        prev = prev.next
>
>    # Reverse from left to right
>    current = prev.next
>    for _ in range(right - left):
>        next_node = current.next
>        current.next = next_node.next
>        next_node.next = prev.next
>        prev.next = next_node
>
>    return dummy.next
>```

>[!example]- JavaScript
>```javascript
>function reverseBetween(head, left, right) {
>    if (!head || left === right) {
>        return head;
>    }
>
>    const dummy = new ListNode(0, head);
>    let prev = dummy;
>
>    // Move to node before left
>    for (let i = 0; i < left - 1; i++) {
>        prev = prev.next;
>    }
>
>    // Reverse from left to right
>    let current = prev.next;
>    for (let i = 0; i < right - left; i++) {
>        const nextNode = current.next;
>        current.next = nextNode.next;
>        nextNode.next = prev.next;
>        prev.next = nextNode;
>    }
>
>    return dummy.next;
>}
>```

### Visualization

```
Reverse positions 2 to 4 in: 1 -> 2 -> 3 -> 4 -> 5

Initial:        1 -> 2 -> 3 -> 4 -> 5
               prev  curr

After step 1:   1 -> 3 -> 2 -> 4 -> 5  (move 3 to front)
               prev       curr

After step 2:   1 -> 4 -> 3 -> 2 -> 5  (move 4 to front)
               prev            curr

Result:         1 -> 4 -> 3 -> 2 -> 5
```

## Reverse in K-Groups

Reverse every k nodes.

>[!example]- C++
>```cpp
>ListNode* reverseKGroup(ListNode* head, int k) {
>    // Check if we have k nodes
>    int count = 0;
>    ListNode* node = head;
>    while (node && count < k) {
>        node = node->next;
>        count++;
>    }
>
>    if (count < k) {
>        return head;  // Not enough nodes
>    }
>
>    // Reverse k nodes
>    ListNode* prev = nullptr;
>    ListNode* current = head;
>    for (int i = 0; i < k; i++) {
>        ListNode* nextNode = current->next;
>        current->next = prev;
>        prev = current;
>        current = nextNode;
>    }
>
>    // head is now the tail of reversed portion
>    // Connect to the rest (recursively reversed)
>    head->next = reverseKGroup(current, k);
>
>    return prev;  // New head of this portion
>}
>```

>[!example]- Java
>```java
>ListNode reverseKGroup(ListNode head, int k) {
>    // Check if we have k nodes
>    int count = 0;
>    ListNode node = head;
>    while (node != null && count < k) {
>        node = node.next;
>        count++;
>    }
>
>    if (count < k) {
>        return head;  // Not enough nodes
>    }
>
>    // Reverse k nodes
>    ListNode prev = null;
>    ListNode current = head;
>    for (int i = 0; i < k; i++) {
>        ListNode nextNode = current.next;
>        current.next = prev;
>        prev = current;
>        current = nextNode;
>    }
>
>    // head is now the tail of reversed portion
>    // Connect to the rest (recursively reversed)
>    head.next = reverseKGroup(current, k);
>
>    return prev;  // New head of this portion
>}
>```

>[!example]- Python
>```python
>def reverseKGroup(head, k):
>    # Check if we have k nodes
>    count = 0
>    node = head
>    while node and count < k:
>        node = node.next
>        count += 1
>
>    if count < k:
>        return head  # Not enough nodes
>
>    # Reverse k nodes
>    prev = None
>    current = head
>    for _ in range(k):
>        next_node = current.next
>        current.next = prev
>        prev = current
>        current = next_node
>
>    # head is now the tail of reversed portion
>    # Connect to the rest (recursively reversed)
>    head.next = reverseKGroup(current, k)
>
>    return prev  # New head of this portion
>```

>[!example]- JavaScript
>```javascript
>function reverseKGroup(head, k) {
>    // Check if we have k nodes
>    let count = 0;
>    let node = head;
>    while (node && count < k) {
>        node = node.next;
>        count++;
>    }
>
>    if (count < k) {
>        return head;  // Not enough nodes
>    }
>
>    // Reverse k nodes
>    let prev = null;
>    let current = head;
>    for (let i = 0; i < k; i++) {
>        const nextNode = current.next;
>        current.next = prev;
>        prev = current;
>        current = nextNode;
>    }
>
>    // head is now the tail of reversed portion
>    // Connect to the rest (recursively reversed)
>    head.next = reverseKGroup(current, k);
>
>    return prev;  // New head of this portion
>}
>```

## Swap Nodes in Pairs

Special case of reverse k-group where k=2.

>[!example]- C++
>```cpp
>ListNode* swapPairs(ListNode* head) {
>    ListNode* dummy = new ListNode(0, head);
>    ListNode* prev = dummy;
>
>    while (prev->next && prev->next->next) {
>        ListNode* first = prev->next;
>        ListNode* second = prev->next->next;
>
>        // Swap
>        prev->next = second;
>        first->next = second->next;
>        second->next = first;
>
>        prev = first;
>    }
>
>    return dummy->next;
>}
>```

>[!example]- Java
>```java
>ListNode swapPairs(ListNode head) {
>    ListNode dummy = new ListNode(0, head);
>    ListNode prev = dummy;
>
>    while (prev.next != null && prev.next.next != null) {
>        ListNode first = prev.next;
>        ListNode second = prev.next.next;
>
>        // Swap
>        prev.next = second;
>        first.next = second.next;
>        second.next = first;
>
>        prev = first;
>    }
>
>    return dummy.next;
>}
>```

>[!example]- Python
>```python
>def swapPairs(head):
>    dummy = ListNode(0, head)
>    prev = dummy
>
>    while prev.next and prev.next.next:
>        first = prev.next
>        second = prev.next.next
>
>        # Swap
>        prev.next = second
>        first.next = second.next
>        second.next = first
>
>        prev = first
>
>    return dummy.next
>```

>[!example]- JavaScript
>```javascript
>function swapPairs(head) {
>    const dummy = new ListNode(0, head);
>    let prev = dummy;
>
>    while (prev.next && prev.next.next) {
>        const first = prev.next;
>        const second = prev.next.next;
>
>        // Swap
>        prev.next = second;
>        first.next = second.next;
>        second.next = first;
>
>        prev = first;
>    }
>
>    return dummy.next;
>}
>```

## Common Patterns

### 1. Using Dummy Node

Always use a dummy node when the head might change:

>[!example]- C++
>```cpp
>ListNode* dummy = new ListNode(0, head);
>// ... operations ...
>return dummy->next;
>```

>[!example]- Java
>```java
>ListNode dummy = new ListNode(0, head);
>// ... operations ...
>return dummy.next;
>```

>[!example]- Python
>```python
>dummy = ListNode(0, head)
># ... operations ...
>return dummy.next
>```

>[!example]- JavaScript
>```javascript
>const dummy = new ListNode(0, head);
>// ... operations ...
>return dummy.next;
>```

### 2. Saving Next Before Modifying

Always save the next pointer before changing links:

>[!example]- C++
>```cpp
>ListNode* nextNode = current->next;  // Save first!
>current->next = prev;                // Then modify
>```

>[!example]- Java
>```java
>ListNode nextNode = current.next;  // Save first!
>current.next = prev;               // Then modify
>```

>[!example]- Python
>```python
>next_node = current.next  # Save first!
>current.next = prev       # Then modify
>```

>[!example]- JavaScript
>```javascript
>const nextNode = current.next;  // Save first!
>current.next = prev;            // Then modify
>```

### 3. Identifying Portions to Reverse

For partial reversal:
1. Find the node **before** the section to reverse
2. Keep track of what will become the **tail** after reversal
3. Connect everything after reversal

## Practice Problems

| Problem | Variation | Difficulty |
|---------|-----------|------------|
| Reverse Linked List | Basic | Easy |
| Reverse Linked List II | Portion | Medium |
| Swap Nodes in Pairs | K=2 | Medium |
| Reverse Nodes in k-Group | K-Group | Hard |
| Palindrome Linked List | Reverse + Compare | Medium |
| Reorder List | Middle + Reverse + Merge | Medium |

## Tips

1. **Draw it out** - Visualize pointer changes
2. **Track all pointers** - prev, current, next, and any anchors
3. **Handle edge cases** - Empty list, single node, exact k nodes
4. **Use dummy node** - When head might change
5. **Test with small lists** - 1, 2, 3 nodes
