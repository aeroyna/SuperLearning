# Stacks and Queues

Stacks and queues are fundamental data structures with restricted access patterns that make them powerful tools for specific problems.

## Overview

- **Stack**: Last-In-First-Out (LIFO)
- **Queue**: First-In-First-Out (FIFO)

## Topics

- [5.1 Stack Fundamentals](Stacks/01_stack_fundamentals.md)
- [5.2 Stack Problems](Stacks/02_stack_problems.md)
- [5.3 Queue Fundamentals](Queues/01_queue_fundamentals.md)
- [5.4 Monotonic Stack](Monotonic_Stack/01_monotonic_stack.md)
- [5.5 Monotonic Queue](Monotonic_Stack/02_monotonic_queue.md)
- [5.6 Practice Problems](Practice_Problems/00_practice_problems.md)

## Stack vs Queue

| Aspect | Stack | Queue |
|--------|-------|-------|
| Access Pattern | LIFO | FIFO |
| Insert | Push (top) | Enqueue (back) |
| Remove | Pop (top) | Dequeue (front) |
| View | Peek (top) | Peek (front) |
| Use Case | Undo, DFS, parsing | BFS, scheduling |

## Implementation

### Stack (using list)

>[!example]- C++
>```cpp
>#include <stack>
>using namespace std;
>
>stack<int> s;
>s.push(1);      // Push
>s.pop();        // Pop
>s.top();        // Peek
>s.empty();      // Is empty
>```

>[!example]- Java
>```java
>import java.util.Stack;
>
>Stack<Integer> stack = new Stack<>();
>stack.push(1);    // Push
>stack.pop();      // Pop
>stack.peek();     // Peek
>stack.isEmpty();  // Is empty
>```

>[!example]- Python
>```python
>stack = []
>stack.append(1)  # Push
>stack.pop()      # Pop
>stack[-1]        # Peek
>len(stack) == 0  # Is empty
>```

>[!example]- JavaScript
>```javascript
>const stack = [];
>stack.push(1);        // Push
>stack.pop();          // Pop
>stack[stack.length - 1]; // Peek
>stack.length === 0;   // Is empty
>```

### Queue (using deque)

>[!example]- C++
>```cpp
>#include <queue>
>using namespace std;
>
>queue<int> q;
>q.push(1);      // Enqueue
>q.pop();        // Dequeue
>q.front();      // Peek
>q.empty();      // Is empty
>```

>[!example]- Java
>```java
>import java.util.LinkedList;
>import java.util.Queue;
>
>Queue<Integer> queue = new LinkedList<>();
>queue.offer(1);    // Enqueue
>queue.poll();      // Dequeue
>queue.peek();      // Peek
>queue.isEmpty();   // Is empty
>```

>[!example]- Python
>```python
>from collections import deque
>queue = deque()
>queue.append(1)    # Enqueue
>queue.popleft()    # Dequeue
>queue[0]           # Peek
>len(queue) == 0    # Is empty
>```

>[!example]- JavaScript
>```javascript
>// Using array (shift is O(n))
>const queue = [];
>queue.push(1);     // Enqueue
>queue.shift();     // Dequeue
>queue[0];          // Peek
>queue.length === 0; // Is empty
>```

## Common Applications

### Stack

1. **Function call management** - Call stack
2. **Undo/Redo operations** - Text editors
3. **Expression evaluation** - Infix, postfix
4. **Parentheses matching** - Valid brackets
5. **DFS implementation** - Graph traversal
6. **Backtracking** - Maze solving
7. **Monotonic stack** - Next greater element

### Queue

1. **BFS implementation** - Graph traversal
2. **Level-order traversal** - Tree problems
3. **Task scheduling** - Process management
4. **Buffering** - Data streaming
5. **Sliding window max** - With monotonic deque

## Key Interview Patterns

| Pattern | Data Structure | Example Problem |
|---------|---------------|-----------------|
| Matching pairs | Stack | Valid Parentheses |
| Next greater/smaller | Monotonic Stack | Daily Temperatures |
| Calculator | Stack | Basic Calculator |
| Level-order | Queue | Binary Tree Level Order |
| Sliding window max | Monotonic Deque | Sliding Window Maximum |
