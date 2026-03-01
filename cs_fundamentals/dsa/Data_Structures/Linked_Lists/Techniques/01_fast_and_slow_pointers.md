# Fast and Slow Pointers

The fast and slow pointers technique (also called Floyd's Tortoise and Hare) uses two pointers moving at different speeds through a linked list.

## Core Concept

- **Slow pointer**: Moves one step at a time
- **Fast pointer**: Moves two steps at a time

This technique is particularly useful for:
1. Finding the middle of a linked list
2. Detecting cycles
3. Finding the start of a cycle

## Pattern 1: Find Middle of Linked List

When fast reaches the end, slow is at the middle.

> [!example]- C++
> ```cpp
> ListNode* findMiddle(ListNode* head) {
>     ListNode* slow = head;
>     ListNode* fast = head;
>
>     while (fast != nullptr && fast->next != nullptr) {
>         slow = slow->next;
>         fast = fast->next->next;
>     }
>
>     return slow;  // Middle node
> }
> ```

> [!example]- Java
> ```java
> public ListNode findMiddle(ListNode head) {
>     ListNode slow = head;
>     ListNode fast = head;
>
>     while (fast != null && fast.next != null) {
>         slow = slow.next;
>         fast = fast.next.next;
>     }
>
>     return slow;  // Middle node
> }
> ```

> [!example]- Python
> ```python
> def findMiddle(head):
>     slow = fast = head
>
>     while fast and fast.next:
>         slow = slow.next
>         fast = fast.next.next
>
>     return slow  # Middle node
> ```

> [!example]- JavaScript
> ```javascript
> function findMiddle(head) {
>     let slow = head;
>     let fast = head;
>
>     while (fast !== null && fast.next !== null) {
>         slow = slow.next;
>         fast = fast.next.next;
>     }
>
>     return slow;  // Middle node
> }
> ```

**Why it works:**
- Fast moves 2x speed of slow
- When fast travels n nodes, slow travels n/2 nodes
- When fast reaches end (n nodes), slow is at middle (n/2)

### For Even Length Lists

```
List: 1 -> 2 -> 3 -> 4 -> None

Step 0: slow=1, fast=1
Step 1: slow=2, fast=3
Step 2: slow=3, fast=None (stops)

Returns: 3 (second middle)
```

To get first middle, check `fast.next.next`:

> [!example]- C++
> ```cpp
> ListNode* findFirstMiddle(ListNode* head) {
>     ListNode* slow = head;
>     ListNode* fast = head;
>
>     while (fast->next != nullptr && fast->next->next != nullptr) {
>         slow = slow->next;
>         fast = fast->next->next;
>     }
>
>     return slow;  // First middle for even length
> }
> ```

> [!example]- Java
> ```java
> public ListNode findFirstMiddle(ListNode head) {
>     ListNode slow = head;
>     ListNode fast = head;
>
>     while (fast.next != null && fast.next.next != null) {
>         slow = slow.next;
>         fast = fast.next.next;
>     }
>
>     return slow;  // First middle for even length
> }
> ```

> [!example]- Python
> ```python
> def findFirstMiddle(head):
>     slow = fast = head
>
>     while fast.next and fast.next.next:
>         slow = slow.next
>         fast = fast.next.next
>
>     return slow  # First middle for even length
> ```

> [!example]- JavaScript
> ```javascript
> function findFirstMiddle(head) {
>     let slow = head;
>     let fast = head;
>
>     while (fast.next !== null && fast.next.next !== null) {
>         slow = slow.next;
>         fast = fast.next.next;
>     }
>
>     return slow;  // First middle for even length
> }
> ```

## Pattern 2: Detect Cycle

If there's a cycle, fast and slow will eventually meet.

> [!example]- C++
> ```cpp
> bool hasCycle(ListNode* head) {
>     ListNode* slow = head;
>     ListNode* fast = head;
>
>     while (fast != nullptr && fast->next != nullptr) {
>         slow = slow->next;
>         fast = fast->next->next;
>
>         if (slow == fast) {
>             return true;
>         }
>     }
>
>     return false;
> }
> ```

> [!example]- Java
> ```java
> public boolean hasCycle(ListNode head) {
>     ListNode slow = head;
>     ListNode fast = head;
>
>     while (fast != null && fast.next != null) {
>         slow = slow.next;
>         fast = fast.next.next;
>
>         if (slow == fast) {
>             return true;
>         }
>     }
>
>     return false;
> }
> ```

> [!example]- Python
> ```python
> def hasCycle(head):
>     slow = fast = head
>
>     while fast and fast.next:
>         slow = slow.next
>         fast = fast.next.next
>
>         if slow == fast:
>             return True
>
>     return False
> ```

> [!example]- JavaScript
> ```javascript
> function hasCycle(head) {
>     let slow = head;
>     let fast = head;
>
>     while (fast !== null && fast.next !== null) {
>         slow = slow.next;
>         fast = fast.next.next;
>
>         if (slow === fast) {
>             return true;
>         }
>     }
>
>     return false;
> }
> ```

**Why it works:**
- If no cycle: fast reaches None
- If cycle exists: fast "catches up" to slow inside the cycle
- Relative speed is 1 node/step, so they must meet

## Pattern 3: Find Cycle Start (Floyd's Algorithm)

After detecting cycle, find where it begins.

> [!example]- C++
> ```cpp
> ListNode* detectCycle(ListNode* head) {
>     ListNode* slow = head;
>     ListNode* fast = head;
>
>     // Phase 1: Detect cycle
>     while (fast != nullptr && fast->next != nullptr) {
>         slow = slow->next;
>         fast = fast->next->next;
>
>         if (slow == fast) {
>             // Phase 2: Find cycle start
>             slow = head;
>             while (slow != fast) {
>                 slow = slow->next;
>                 fast = fast->next;
>             }
>             return slow;  // Cycle start
>         }
>     }
>
>     return nullptr;  // No cycle
> }
> ```

> [!example]- Java
> ```java
> public ListNode detectCycle(ListNode head) {
>     ListNode slow = head;
>     ListNode fast = head;
>
>     // Phase 1: Detect cycle
>     while (fast != null && fast.next != null) {
>         slow = slow.next;
>         fast = fast.next.next;
>
>         if (slow == fast) {
>             // Phase 2: Find cycle start
>             slow = head;
>             while (slow != fast) {
>                 slow = slow.next;
>                 fast = fast.next;
>             }
>             return slow;  // Cycle start
>         }
>     }
>
>     return null;  // No cycle
> }
> ```

> [!example]- Python
> ```python
> def detectCycle(head):
>     slow = fast = head
>
>     # Phase 1: Detect cycle
>     while fast and fast.next:
>         slow = slow.next
>         fast = fast.next.next
>
>         if slow == fast:
>             break
>     else:
>         return None  # No cycle
>
>     # Phase 2: Find cycle start
>     slow = head
>     while slow != fast:
>         slow = slow.next
>         fast = fast.next
>
>     return slow  # Cycle start
> ```

> [!example]- JavaScript
> ```javascript
> function detectCycle(head) {
>     let slow = head;
>     let fast = head;
>
>     // Phase 1: Detect cycle
>     while (fast !== null && fast.next !== null) {
>         slow = slow.next;
>         fast = fast.next.next;
>
>         if (slow === fast) {
>             // Phase 2: Find cycle start
>             slow = head;
>             while (slow !== fast) {
>                 slow = slow.next;
>                 fast = fast.next;
>             }
>             return slow;  // Cycle start
>         }
>     }
>
>     return null;  // No cycle
> }
> ```

**Mathematical Proof:**
- Let `a` = distance from head to cycle start
- Let `b` = distance from cycle start to meeting point
- Let `c` = cycle length

When they meet:
- Slow traveled: `a + b`
- Fast traveled: `a + b + nc` (n complete cycles)
- Fast travels 2x slow: `2(a + b) = a + b + nc`
- Therefore: `a + b = nc`, so `a = nc - b`

This means: distance from head to cycle start equals distance from meeting point to cycle start (going around the cycle).

## Pattern 4: Find Cycle Length

> [!example]- C++
> ```cpp
> int cycleLength(ListNode* head) {
>     ListNode* slow = head;
>     ListNode* fast = head;
>
>     // Detect cycle
>     while (fast != nullptr && fast->next != nullptr) {
>         slow = slow->next;
>         fast = fast->next->next;
>
>         if (slow == fast) {
>             // Count cycle length
>             int count = 1;
>             ListNode* current = slow->next;
>             while (current != slow) {
>                 count++;
>                 current = current->next;
>             }
>             return count;
>         }
>     }
>
>     return 0;  // No cycle
> }
> ```

> [!example]- Java
> ```java
> public int cycleLength(ListNode head) {
>     ListNode slow = head;
>     ListNode fast = head;
>
>     // Detect cycle
>     while (fast != null && fast.next != null) {
>         slow = slow.next;
>         fast = fast.next.next;
>
>         if (slow == fast) {
>             // Count cycle length
>             int count = 1;
>             ListNode current = slow.next;
>             while (current != slow) {
>                 count++;
>                 current = current.next;
>             }
>             return count;
>         }
>     }
>
>     return 0;  // No cycle
> }
> ```

> [!example]- Python
> ```python
> def cycleLength(head):
>     slow = fast = head
>
>     # Detect cycle
>     while fast and fast.next:
>         slow = slow.next
>         fast = fast.next.next
>         if slow == fast:
>             break
>     else:
>         return 0  # No cycle
>
>     # Count cycle length
>     count = 1
>     current = slow.next
>     while current != slow:
>         count += 1
>         current = current.next
>
>     return count
> ```

> [!example]- JavaScript
> ```javascript
> function cycleLength(head) {
>     let slow = head;
>     let fast = head;
>
>     // Detect cycle
>     while (fast !== null && fast.next !== null) {
>         slow = slow.next;
>         fast = fast.next.next;
>
>         if (slow === fast) {
>             // Count cycle length
>             let count = 1;
>             let current = slow.next;
>             while (current !== slow) {
>                 count++;
>                 current = current.next;
>             }
>             return count;
>         }
>     }
>
>     return 0;  // No cycle
> }
> ```

## Pattern 5: Find Nth Node from End

Use two pointers, n nodes apart.

> [!example]- C++
> ```cpp
> ListNode* removeNthFromEnd(ListNode* head, int n) {
>     ListNode* dummy = new ListNode(0, head);
>     ListNode* slow = dummy;
>     ListNode* fast = dummy;
>
>     // Move fast n+1 steps ahead
>     for (int i = 0; i <= n; i++) {
>         fast = fast->next;
>     }
>
>     // Move both until fast reaches end
>     while (fast != nullptr) {
>         slow = slow->next;
>         fast = fast->next;
>     }
>
>     // slow is now at node before target
>     slow->next = slow->next->next;
>
>     return dummy->next;
> }
> ```

> [!example]- Java
> ```java
> public ListNode removeNthFromEnd(ListNode head, int n) {
>     ListNode dummy = new ListNode(0, head);
>     ListNode slow = dummy;
>     ListNode fast = dummy;
>
>     // Move fast n+1 steps ahead
>     for (int i = 0; i <= n; i++) {
>         fast = fast.next;
>     }
>
>     // Move both until fast reaches end
>     while (fast != null) {
>         slow = slow.next;
>         fast = fast.next;
>     }
>
>     // slow is now at node before target
>     slow.next = slow.next.next;
>
>     return dummy.next;
> }
> ```

> [!example]- Python
> ```python
> def removeNthFromEnd(head, n):
>     dummy = ListNode(0, head)
>     slow = fast = dummy
>
>     # Move fast n+1 steps ahead
>     for _ in range(n + 1):
>         fast = fast.next
>
>     # Move both until fast reaches end
>     while fast:
>         slow = slow.next
>         fast = fast.next
>
>     # slow is now at node before target
>     slow.next = slow.next.next
>
>     return dummy.next
> ```

> [!example]- JavaScript
> ```javascript
> function removeNthFromEnd(head, n) {
>     const dummy = new ListNode(0, head);
>     let slow = dummy;
>     let fast = dummy;
>
>     // Move fast n+1 steps ahead
>     for (let i = 0; i <= n; i++) {
>         fast = fast.next;
>     }
>
>     // Move both until fast reaches end
>     while (fast !== null) {
>         slow = slow.next;
>         fast = fast.next;
>     }
>
>     // slow is now at node before target
>     slow.next = slow.next.next;
>
>     return dummy.next;
> }
> ```

## Pattern 6: Check Palindrome

1. Find middle
2. Reverse second half
3. Compare halves

> [!example]- C++
> ```cpp
> bool isPalindrome(ListNode* head) {
>     // Find middle
>     ListNode* slow = head;
>     ListNode* fast = head;
>     while (fast != nullptr && fast->next != nullptr) {
>         slow = slow->next;
>         fast = fast->next->next;
>     }
>
>     // Reverse second half
>     ListNode* prev = nullptr;
>     while (slow != nullptr) {
>         ListNode* nextNode = slow->next;
>         slow->next = prev;
>         prev = slow;
>         slow = nextNode;
>     }
>
>     // Compare halves
>     ListNode* left = head;
>     ListNode* right = prev;
>     while (right != nullptr) {
>         if (left->val != right->val) {
>             return false;
>         }
>         left = left->next;
>         right = right->next;
>     }
>
>     return true;
> }
> ```

> [!example]- Java
> ```java
> public boolean isPalindrome(ListNode head) {
>     // Find middle
>     ListNode slow = head;
>     ListNode fast = head;
>     while (fast != null && fast.next != null) {
>         slow = slow.next;
>         fast = fast.next.next;
>     }
>
>     // Reverse second half
>     ListNode prev = null;
>     while (slow != null) {
>         ListNode nextNode = slow.next;
>         slow.next = prev;
>         prev = slow;
>         slow = nextNode;
>     }
>
>     // Compare halves
>     ListNode left = head;
>     ListNode right = prev;
>     while (right != null) {
>         if (left.val != right.val) {
>             return false;
>         }
>         left = left.next;
>         right = right.next;
>     }
>
>     return true;
> }
> ```

> [!example]- Python
> ```python
> def isPalindrome(head):
>     # Find middle
>     slow = fast = head
>     while fast and fast.next:
>         slow = slow.next
>         fast = fast.next.next
>
>     # Reverse second half
>     prev = None
>     while slow:
>         next_node = slow.next
>         slow.next = prev
>         prev = slow
>         slow = next_node
>
>     # Compare halves
>     left, right = head, prev
>     while right:
>         if left.val != right.val:
>             return False
>         left = left.next
>         right = right.next
>
>     return True
> ```

> [!example]- JavaScript
> ```javascript
> function isPalindrome(head) {
>     // Find middle
>     let slow = head;
>     let fast = head;
>     while (fast !== null && fast.next !== null) {
>         slow = slow.next;
>         fast = fast.next.next;
>     }
>
>     // Reverse second half
>     let prev = null;
>     while (slow !== null) {
>         const nextNode = slow.next;
>         slow.next = prev;
>         prev = slow;
>         slow = nextNode;
>     }
>
>     // Compare halves
>     let left = head;
>     let right = prev;
>     while (right !== null) {
>         if (left.val !== right.val) {
>             return false;
>         }
>         left = left.next;
>         right = right.next;
>     }
>
>     return true;
> }
> ```

## Common Mistakes

1. **Not checking `fast.next`** before accessing `fast.next.next`
2. **Off-by-one** when finding middle of even-length lists
3. **Not handling empty list** or single node
4. **Forgetting to restore** list structure after modification

## Practice Problems

| Problem | Technique | Difficulty |
|---------|-----------|------------|
| Middle of Linked List | Basic | Easy |
| Linked List Cycle | Detect | Easy |
| Linked List Cycle II | Find Start | Medium |
| Palindrome Linked List | Full Pattern | Medium |
| Remove Nth From End | Distance | Medium |
| Reorder List | Middle + Reverse | Medium |
