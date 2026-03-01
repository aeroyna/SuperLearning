## Stack Fundamentals (LIFO)

A stack is a linear data structure that follows the **LIFO (Last-In, First-Out)** principle. The last element added to the stack is always the first one to be removed.

### Core Idea & Analogy
Think of a stack of plates or a pile of books. You can only add a new item to the top of the stack, and you can only remove the topmost item. This simple restriction is surprisingly powerful. In computer science, the most direct analogy is the "call stack" that manages function calls in a program.

### Key Operations
A stack supports three main operations, all of which should ideally be O(1).
- **Push**: Add an element to the top of the stack.
- **Pop**: Remove and return the element from the top of the stack.
- **Peek** (or Top): Look at the top element without removing it.

### Implementation

>[!example]- C++
>```cpp
>// Initialize an empty stack
>#include <stack>
>#include <vector>
>
>std::stack<int> stack;
>
>// Push operations
>stack.push(10);  // stack has 10
>stack.push(20);  // stack has 10, 20
>stack.push(30);  // stack has 10, 20, 30
>
>// Peek operation
>if (!stack.empty()) {
>    int top_element = stack.top();
>    // top_element is 30
>}
>
>// Pop operation
>if (!stack.empty()) {
>    int removed_element = stack.top();
>    stack.pop();
>    // removed_element is 30, stack now has 10, 20
>}
>
>// Check if empty
>bool is_empty = stack.empty(); // false
>```

>[!example]- Java
>```java
>// Initialize an empty stack
>import java.util.Stack;
>
>Stack<Integer> stack = new Stack<>();
>
>// Push operations
>stack.push(10);  // stack is [10]
>stack.push(20);  // stack is [10, 20]
>stack.push(30);  // stack is [10, 20, 30]
>
>// Peek operation
>if (!stack.isEmpty()) {
>    int topElement = stack.peek();
>    // topElement is 30
>}
>
>// Pop operation
>if (!stack.isEmpty()) {
>    int removedElement = stack.pop();
>    // removedElement is 30, stack is now [10, 20]
>}
>
>// Check if empty
>boolean isEmpty = stack.isEmpty(); // false
>```

>[!example]- Python
>```python
># Initialize an empty stack
>stack = []
>
># Push operations
>stack.append(10)  # stack is [10]
>stack.append(20)  # stack is [10, 20]
>stack.append(30)  # stack is [10, 20, 30]
>
># Peek operation
>if stack:
>    top_element = stack[-1]
>    # top_element is 30
>
># Pop operation
>if stack:
>    removed_element = stack.pop()
>    # removed_element is 30, stack is now [10, 20]
>
># Check if empty
>is_empty = not stack # False
>```

>[!example]- JavaScript
>```javascript
>// Initialize an empty stack
>const stack = [];
>
>// Push operations
>stack.push(10);  // stack is [10]
>stack.push(20);  // stack is [10, 20]
>stack.push(30);  // stack is [10, 20, 30]
>
>// Peek operation
>if (stack.length > 0) {
>    const topElement = stack[stack.length - 1];
>    // topElement is 30
>}
>
>// Pop operation
>if (stack.length > 0) {
>    const removedElement = stack.pop();
>    // removedElement is 30, stack is now [10, 20]
>}
>
>// Check if empty
>const isEmpty = stack.length === 0; // false
>```

### Implementation Notes by Language

**Python**: A standard Python `list` serves as an excellent and efficient stack.
- `list.append()` is the **push** operation.
- `list.pop()` (with no index) is the **pop** operation.
- Accessing the last element (`my_stack[-1]`) is the **peek** operation.
- All these operations have an amortized O(1) time complexity.

**C++**: Use `std::stack` from the `<stack>` header.
- `stack.push()` adds to the top.
- `stack.pop()` removes from the top (returns void).
- `stack.top()` peeks at the top element.
- Always check `!stack.empty()` before accessing.

**Java**: Use `Stack<T>` from `java.util`.
- `stack.push()` adds to the top.
- `stack.pop()` removes and returns the top element.
- `stack.peek()` looks at the top without removing.
- `stack.isEmpty()` checks if empty.

**JavaScript**: Use arrays with push/pop methods.
- `array.push()` adds to the end.
- `array.pop()` removes and returns from the end.
- `array[array.length - 1]` peeks at the top.
- Check `array.length > 0` before accessing.
