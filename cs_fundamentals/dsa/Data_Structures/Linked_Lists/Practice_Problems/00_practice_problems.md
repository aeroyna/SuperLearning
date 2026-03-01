# Practice Problems: Linked Lists

Focus on pointer manipulation, dummy nodes, and slow/fast pointer techniques.

## Basic Operations

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/) | Easy | Track `prev`, `curr`, `next`. `curr.next = prev`. |
| [Merge Two Sorted Lists](https://leetcode.com/problems/merge-two-sorted-lists/) | Easy | Use a dummy head to simplify edge cases. |

## Fast & Slow Pointers

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Linked List Cycle](https://leetcode.com/problems/linked-list-cycle/) | Easy | If `fast` and `slow` meet, there's a cycle. |
| [Linked List Cycle II](https://leetcode.com/problems/linked-list-cycle-ii/) | Medium | After meeting, reset `slow` to head. Meeting point is cycle start. |
| [Middle of the Linked List](https://leetcode.com/problems/middle-of-the-linked-list/) | Easy | `fast` moves 2x speed. When `fast` ends, `slow` is middle. |
| [Remove Nth Node From End of List](https://leetcode.com/problems/remove-nth-node-from-end-of-list/) | Medium | Move `fast` n steps ahead, then move both until `fast` ends. |

## Advanced

| Problem | Difficulty | Key Insight |
|---------|------------|-------------|
| [Merge k Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/) | Hard | Min-heap of head nodes or divide-and-conquer merge. |
| [Reverse Nodes in k-Group](https://leetcode.com/problems/reverse-nodes-in-k-group/) | Hard | Check if k nodes exist, reverse them, recurse/iterate. |
| [Copy List with Random Pointer](https://leetcode.com/problems/copy-list-with-random-pointer/) | Medium | Interweave new nodes `A->A'->B->B'`, set randoms, then separate. |
