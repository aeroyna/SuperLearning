# Stacks

A stack is a Last-In-First-Out (LIFO) data structure that restricts access to only the most recently added element. This constraint, rather than being limiting, enables elegant solutions for a specific class of problems.

## Overview

Stacks support three primary operations:
- **Push**: Add element to top - O(1)
- **Pop**: Remove element from top - O(1)
- **Peek**: View top element without removal - O(1)

## Topics

- [5.1.1 Stack Fundamentals](01_stack_fundamentals.md)
- [5.1.2 Stack Problems](02_stack_problems.md)

## Internal Implementation

### Array-Based Stack

>[!example]- C++
>```cpp
>class ArrayStack {
>    vector<int> data;
>public:
>    void push(int val) {
>        data.push_back(val);
>    }
>    int pop() {
>        if (data.empty()) throw runtime_error("Stack empty");
>        int val = data.back();
>        data.pop_back();
>        return val;
>    }
>    int top() {
>        if (data.empty()) throw runtime_error("Stack empty");
>        return data.back();
>    }
>    bool empty() {
>        return data.empty();
>    }
>};
>```

>[!example]- Java
>```java
>class ArrayStack {
>    private List<Integer> data = new ArrayList<>();
>    
>    public void push(int val) {
>        data.add(val);
>    }
>    public int pop() {
>        if (data.isEmpty()) throw new EmptyStackException();
>        return data.remove(data.size() - 1);
>    }
>    public int peek() {
>        if (data.isEmpty()) throw new EmptyStackException();
>        return data.get(data.size() - 1);
>    }
>    public boolean isEmpty() {
>        return data.isEmpty();
>    }
>}
>```

>[!example]- Python
>```python
>class ArrayStack:
>    def __init__(self, capacity=16):
>        self._data = [None] * capacity
>        self._top = -1
>
>    def push(self, val):
>        if self._top == len(self._data) - 1:
>            self._resize(2 * len(self._data))
>        self._top += 1
>        self._data[self._top] = val
>
>    def pop(self):
>        if self._top == -1:
>            raise IndexError("Stack empty")
>        val = self._data[self._top]
>        self._data[self._top] = None  # Help GC
>        self._top -= 1
>        return val
>
>    def _resize(self, new_capacity):
>        new_data = [None] * new_capacity
>        for i in range(self._top + 1):
>            new_data[i] = self._data[i]
>        self._data = new_data
>```

>[!example]- JavaScript
>```javascript
>class ArrayStack {
>    constructor() {
>        this.data = [];
>    }
>    push(val) {
>        this.data.push(val);
>    }
>    pop() {
>        if (this.data.length === 0) throw new Error("Stack empty");
>        return this.data.pop();
>    }
>    peek() {
>        if (this.data.length === 0) throw new Error("Stack empty");
>        return this.data[this.data.length - 1];
>    }
>    isEmpty() {
>        return this.data.length === 0;
>    }
>}
>```

**Memory layout**:
```
_data:  [elem0 | elem1 | elem2 | None | None | ...]
                         ^
                        _top = 2

Push: _top++, then assign
Pop:  read at _top, then _top--
```

**Amortized analysis**: Resizing doubles capacity, so each element causes at most O(log n) resizes. Amortized push is O(1).

### Linked List-Based Stack

```python
class LinkedStack:
    def __init__(self):
        self._head = None
        self._size = 0

    def push(self, val):
        self._head = Node(val, self._head)
        self._size += 1

    def pop(self):
        if not self._head:
            raise IndexError("Stack empty")
        val = self._head.val
        self._head = self._head.next
        self._size -= 1
        return val
```

**Trade-offs**:
| Aspect | Array-Based | Linked-Based |
|--------|-------------|--------------|
| Cache locality | Excellent | Poor |
| Memory overhead | Low (occasional resize) | High (pointer per element) |
| Worst-case push | O(n) resize | O(1) always |
| In practice | Usually faster | More predictable |

## The Call Stack

Understanding the program's call stack illuminates why recursion and stack-based iteration are interchangeable:

```python
def factorial(n):  # Each call creates a stack frame
    if n <= 1:
        return 1
    return n * factorial(n - 1)

# Iterative with explicit stack
def factorial_iter(n):
    stack = []
    result = 1
    while n > 1:
        stack.append(n)
        n -= 1
    while stack:
        result *= stack.pop()
    return result
```

**Stack frame contents**:
- Return address
- Local variables
- Parameters
- Previous frame pointer

## Common Stack Patterns

### Pattern 1: Matching Pairs

```python
def is_valid_parentheses(s):
    stack = []
    pairs = {')': '(', '}': '{', ']': '['}
    for char in s:
        if char in '({[':
            stack.append(char)
        elif char in ')}]':
            if not stack or stack[-1] != pairs[char]:
                return False
            stack.pop()
    return len(stack) == 0
```

### Pattern 2: Expression Evaluation

```python
def eval_rpn(tokens):
    stack = []
    ops = {'+': lambda a, b: a + b,
           '-': lambda a, b: a - b,
           '*': lambda a, b: a * b,
           '/': lambda a, b: int(a / b)}

    for token in tokens:
        if token in ops:
            b, a = stack.pop(), stack.pop()
            stack.append(ops[token](a, b))
        else:
            stack.append(int(token))
    return stack[0]
```

### Pattern 3: Converting Recursion to Iteration

Any recursive algorithm can be converted to iteration using an explicit stack:

```python
# DFS recursive
def dfs_recursive(node):
    if not node:
        return
    process(node)
    dfs_recursive(node.left)
    dfs_recursive(node.right)

# DFS iterative
def dfs_iterative(root):
    stack = [root]
    while stack:
        node = stack.pop()
        if not node:
            continue
        process(node)
        stack.append(node.right)  # Push right first (LIFO)
        stack.append(node.left)
```

## Common Pitfalls

1. **Empty stack access**: Always check `if stack` before `stack.pop()` or `stack[-1]`
2. **Using list as stack incorrectly**: Use `append/pop`, not `insert(0)/pop(0)` (that's O(n))
3. **Order in expression evaluation**: For non-commutative ops, order matters: `a - b ≠ b - a`
4. **Stack overflow**: Deep recursion can exceed call stack limits (typically ~1000 in Python)

## Key Interview Problems

| Problem | Pattern | Difficulty | LeetCode Link |
| --------- | --------- | ------------ | --- |
| Valid Parentheses | Matching | Easy | [Link](https://leetcode.com/problems/valid-parentheses/) |
| Min Stack | Auxiliary stack | Medium | [Link](https://leetcode.com/problems/min-stack/) |
| Evaluate RPN | Expression | Medium | [Link](https://leetcode.com/problems/evaluate-reverse-polish-notation/) |
| Basic Calculator | Expression + precedence | Hard | [Link](https://leetcode.com/problems/basic-calculator/) |
| Decode String | Nested structure | Medium | [Link](https://leetcode.com/problems/decode-string/) |
| Largest Rectangle in Histogram | Monotonic stack | Hard | [Link](https://leetcode.com/problems/largest-rectangle-in-histogram/) |
