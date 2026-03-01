# Linked List Techniques

Linked list problems require specific manipulation techniques due to the node-based structure. These techniques form the foundation for solving most linked list interview problems efficiently.

## Overview

Core techniques for linked list manipulation:
1. **Fast and Slow Pointers** - Detecting cycles, finding midpoints
2. **Reversal** - In-place list reversal, partial reversal
3. **Dummy Nodes** - Simplifying edge cases
4. **Merge Techniques** - Combining sorted lists

## Topics

- [4.2.1 Fast and Slow Pointers](01_fast_and_slow_pointers.md)
- [4.2.2 Reversing Linked Lists](02_reversing_linked_lists.md)

## Technique 1: Fast and Slow Pointers (Floyd's Algorithm)

Two pointers moving at different speeds to detect cycles or find positions.

### Cycle Detection

>[!example]- C++
>```cpp
>bool hasCycle(ListNode *head) {
>    ListNode *slow = head, *fast = head;
>    while (fast && fast->next) {
>        slow = slow->next;
>        fast = fast->next->next;
>        if (slow == fast) return true;
>    }
>    return false;
>}
>```

>[!example]- Java
>```java
>public boolean hasCycle(ListNode head) {
>    ListNode slow = head, fast = head;
>    while (fast != null && fast.next != null) {
>        slow = slow.next;
>        fast = fast.next.next;
>        if (slow == fast) return true;
>    }
>    return false;
>}
>```

>[!example]- Python
>```python
>def has_cycle(head):
>    slow = fast = head
>    while fast and fast.next:
>        slow = slow.next
>        fast = fast.next.next
>        if slow == fast:
>            return True
>    return False
>```

>[!example]- JavaScript
>```javascript
>function hasCycle(head) {
>    let slow = head, fast = head;
>    while (fast && fast.next) {
>        slow = slow.next;
>        fast = fast.next.next;
>        if (slow === fast) return true;
>    }
>    return false;
>}
>```

**Why it works**: If there's a cycle, fast will eventually lap slow. If no cycle, fast reaches null. Mathematical proof: fast gains one node per iteration, so they must meet within cycle length iterations.

### Finding Cycle Start

>[!example]- C++
>```cpp
>ListNode *detectCycle(ListNode *head) {
>    ListNode *slow = head, *fast = head;
>    while (fast && fast->next) {
>        slow = slow->next;
>        fast = fast->next->next;
>        if (slow == fast) {
>            slow = head;
>            while (slow != fast) {
>                slow = slow->next;
>                fast = fast->next;
>            }
>            return slow;
>        }
>    }
>    return nullptr;
>}
>```

>[!example]- Java
>```java
>public ListNode detectCycle(ListNode head) {
>    ListNode slow = head, fast = head;
>    while (fast != null && fast.next != null) {
>        slow = slow.next;
>        fast = fast.next.next;
>        if (slow == fast) {
>            slow = head;
>            while (slow != fast) {
>                slow = slow.next;
>                fast = fast.next;
>            }
>            return slow;
>        }
>    }
>    return null;
>}
>```

>[!example]- Python
>```python
>def detect_cycle_start(head):
>    slow = fast = head
>    while fast and fast.next:
>        slow = slow.next
>        fast = fast.next.next
>        if slow == fast:
>            # Reset slow to head, move both at same speed
>            slow = head
>            while slow != fast:
>                slow = slow.next
>                fast = fast.next
>            return slow
>    return None
>```

>[!example]- JavaScript
>```javascript
>function detectCycle(head) {
>    let slow = head, fast = head;
>    while (fast && fast.next) {
>        slow = slow.next;
>        fast = fast.next.next;
>        if (slow === fast) {
>            slow = head;
>            while (slow !== fast) {
>                slow = slow.next;
>                fast = fast.next;
>            }
>            return slow;
>        }
>    }
>    return null;
>}
>```

**Mathematical insight**: Let distance to cycle start = a, cycle length = c. When they meet, slow traveled a + b, fast traveled a + b + nc. Since fast = 2*slow: a + b + nc = 2(a + b), so nc = a + b, meaning a = nc - b. Starting from meeting point, traveling a more steps lands on cycle start.

### Finding Middle Node

>[!example]- C++
>```cpp
>ListNode* middleNode(ListNode* head) {
>    ListNode *slow = head, *fast = head;
>    while (fast && fast->next) {
>        slow = slow->next;
>        fast = fast->next->next;
>    }
>    return slow;
>}
>```

>[!example]- Java
>```java
>public ListNode middleNode(ListNode head) {
>    ListNode slow = head, fast = head;
>    while (fast != null && fast.next != null) {
>        slow = slow.next;
>        fast = fast.next.next;
>    }
>    return slow;
>}
>```

>[!example]- Python
>```python
>def find_middle(head):
>    slow = fast = head
>    while fast and fast.next:
>        slow = slow.next
>        fast = fast.next.next
>    return slow  # For odd length: exact middle; for even: second middle
>```

>[!example]- JavaScript
>```javascript
>function middleNode(head) {
>    let slow = head, fast = head;
>    while (fast && fast.next) {
>        slow = slow.next;
>        fast = fast.next.next;
>    }
>    return slow;
>}
>```

## Technique 2: In-Place Reversal

>[!example]- C++
>```cpp
>ListNode* reverseList(ListNode* head) {
>    ListNode *prev = nullptr, *curr = head;
>    while (curr) {
>        ListNode *nextTemp = curr->next;
>        curr->next = prev;
>        prev = curr;
>        curr = nextTemp;
>    }
>    return prev;
>}
>```

>[!example]- Java
>```java
>public ListNode reverseList(ListNode head) {
>    ListNode prev = null, curr = head;
>    while (curr != null) {
>        ListNode nextTemp = curr.next;
>        curr.next = prev;
>        prev = curr;
>        curr = nextTemp;
>    }
>    return prev;
>}
>```

>[!example]- Python
>```python
>def reverse_list(head):
>    prev = None
>    curr = head
>    while curr:
>        next_temp = curr.next  # Save next
>        curr.next = prev       # Reverse pointer
>        prev = curr            # Move prev forward
>        curr = next_temp       # Move curr forward
>    return prev
>```

>[!example]- JavaScript
>```javascript
>function reverseList(head) {
>    let prev = null, curr = head;
>    while (curr) {
>        const nextTemp = curr.next;
>        curr.next = prev;
>        prev = curr;
>        curr = nextTemp;
>    }
>    return prev;
>}
>```

**Pointer state visualization**:
```
Initial:  null   1 → 2 → 3 → null
          prev  curr

Step 1:   null ← 1   2 → 3 → null
                prev curr

Step 2:   null ← 1 ← 2   3 → null
                    prev curr

Step 3:   null ← 1 ← 2 ← 3   null
                        prev curr

Final head = prev
```

### Partial Reversal (Between Positions)

>[!example]- C++
>```cpp
>ListNode* reverseBetween(ListNode* head, int left, int right) {
>    if (!head || left == right) return head;
>    
>    ListNode dummy(0, head);
>    ListNode *prev = &dummy;
>    
>    for (int i = 0; i < left - 1; i++) prev = prev->next;
>    
>    ListNode *curr = prev->next;
>    for (int i = 0; i < right - left; i++) {
>        ListNode *nextTemp = curr->next;
>        curr->next = nextTemp->next;
>        nextTemp->next = prev->next;
>        prev->next = nextTemp;
>    }
>    return dummy.next;
>}
>```

>[!example]- Java
>```java
>public ListNode reverseBetween(ListNode head, int left, int right) {
>    if (head == null || left == right) return head;
>    
>    ListNode dummy = new ListNode(0, head);
>    ListNode prev = dummy;
>    
>    for (int i = 0; i < left - 1; i++) prev = prev.next;
>    
>    ListNode curr = prev.next;
>    for (int i = 0; i < right - left; i++) {
>        ListNode nextTemp = curr.next;
>        curr.next = nextTemp.next;
>        nextTemp.next = prev.next;
>        prev.next = nextTemp;
>    }
>    return dummy.next;
>}
>```

>[!example]- Python
>```python
>def reverse_between(head, left, right):
>    if not head or left == right:
>        return head
>
>    dummy = ListNode(0, head)
>    prev = dummy
>
>    # Move to position before left
>    for _ in range(left - 1):
>        prev = prev.next
>
>    # Reverse from left to right
>    curr = prev.next
>    for _ in range(right - left):
>        next_node = curr.next
>        curr.next = next_node.next
>        next_node.next = prev.next
>        prev.next = next_node
>
>    return dummy.next
>```

>[!example]- JavaScript
>```javascript
>function reverseBetween(head, left, right) {
>    if (!head || left === right) return head;
>    
>    const dummy = new ListNode(0, head);
>    let prev = dummy;
>    
>    for (let i = 0; i < left - 1; i++) prev = prev.next;
>    
>    let curr = prev.next;
>    for (let i = 0; i < right - left; i++) {
>        const nextNode = curr.next;
>        curr.next = nextNode.next;
>        nextNode.next = prev.next;
>        prev.next = nextNode;
>    }
>    return dummy.next;
>}
>```

## Technique 3: Dummy Node Pattern

>[!example]- C++
>```cpp
>ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
>    ListNode dummy(0);
>    ListNode *tail = &dummy;
>    
>    while (l1 && l2) {
>        if (l1->val <= l2->val) {
>            tail->next = l1;
>            l1 = l1->next;
>        } else {
>            tail->next = l2;
>            l2 = l2->next;
>        }
>        tail = tail->next;
>    }
>    tail->next = l1 ? l1 : l2;
>    return dummy.next;
>}
>```

>[!example]- Java
>```java
>public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
>    ListNode dummy = new ListNode(0);
>    ListNode tail = dummy;
>    
>    while (l1 != null && l2 != null) {
>        if (l1.val <= l2.val) {
>            tail.next = l1;
>            l1 = l1.next;
>        } else {
>            tail.next = l2;
>            l2 = l2.next;
>        }
>        tail = tail.next;
>    }
>    tail.next = (l1 != null) ? l1 : l2;
>    return dummy.next;
>}
>```

>[!example]- Python
>```python
>def merge_two_lists(l1, l2):
>    dummy = ListNode(0)
>    tail = dummy
>
>    while l1 and l2:
>        if l1.val <= l2.val:
>            tail.next = l1
>            l1 = l1.next
>        else:
>            tail.next = l2
>            l2 = l2.next
>        tail = tail.next
>
>    tail.next = l1 or l2
>    return dummy.next
>```

>[!example]- JavaScript
>```javascript
>function mergeTwoLists(l1, l2) {
>    const dummy = new ListNode(0);
>    let tail = dummy;
>    
>    while (l1 && l2) {
>        if (l1.val <= l2.val) {
>            tail.next = l1;
>            l1 = l1.next;
>        } else {
>            tail.next = l2;
>            l2 = l2.next;
>        }
>        tail = tail.next;
>    }
>    tail.next = l1 || l2;
>    return dummy.next;
>}
>```

**Why dummy helps**: Without it, we'd need special logic for the first node. Dummy provides a stable anchor regardless of which list starts the result.

## Common Pitfalls

1. **Losing node references**: Save `next` before modifying pointers
2. **Null pointer access**: Always check `node and node.next` before accessing `node.next.next`
3. **Forgetting edge cases**: Empty list, single node, head changes
4. **Off-by-one in positioning**: List positions are typically 1-indexed in problems

## Complexity Analysis

| Technique | Time | Space |
|-----------|------|-------|
| Cycle detection | O(n) | O(1) |
| Find middle | O(n) | O(1) |
| Reverse list | O(n) | O(1) |
| Merge sorted lists | O(n + m) | O(1) |

## Key Interview Problems

| Problem | Technique | Difficulty | LeetCode Link |
| --------- | ----------- | ------------ | --- |
| Linked List Cycle | Fast/Slow | Easy | [Link](https://leetcode.com/problems/linked-list-cycle/) |
| Middle of Linked List | Fast/Slow | Easy | [Link](https://leetcode.com/problems/middle-of-linked-list/) |
| Reverse Linked List | Reversal | Easy | [Link](https://leetcode.com/problems/reverse-linked-list/) |
| Palindrome Linked List | Middle + Reverse | Easy | [Link](https://leetcode.com/problems/palindrome-linked-list/) |
| Reorder List | Middle + Reverse + Merge | Medium | [Link](https://leetcode.com/problems/reorder-list/) |
| Reverse Nodes in k-Group | Partial Reversal | Hard | [Link](https://leetcode.com/problems/reverse-nodes-in-k-group/) |
