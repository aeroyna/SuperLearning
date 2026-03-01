## Common Stack Problems and Patterns

Stacks are versatile and appear in a variety of problem patterns. Understanding these patterns is key to recognizing when to use a stack.

### Pattern 1: Matching/Validating Pairs

This is the most common stack pattern, epitomized by the "Valid Parentheses" problem. It's used whenever you need to validate or process nested structures.

#### Example: Valid Parentheses (LeetCode #20)

**Problem**: Given a string `s` containing just the characters `(`, `)`, `{`, `}`, `[` and `]`, determine if the input string is valid. An input string is valid if:
1. Open brackets must be closed by the same type of brackets.
2. Open brackets must be closed in the correct order.

**Solution**:
The LIFO nature of a stack is perfect for this. When we see an opening bracket, we "push" it onto the stack, saving it for later. When we see a closing bracket, we "pop" from the stack to see if the last-opened bracket is the correct matching pair.

1.  Initialize an empty stack.
2.  Create a hash map to store the matching pairs (e.g., `')': '('`).
3.  Iterate through the input string:
    -   If the character is an **opening** bracket, `push` it onto the stack.
    -   If the character is a **closing** bracket:
        -   Check if the stack is empty. If it is, there's no matching opening bracket, so it's invalid.
        -   `pop` the top element from the stack.
        -   Check if the popped element is the correct opening bracket for the current closing bracket. If not, it's invalid.
4.  After the loop, if the stack is **empty**, it means every opening bracket had a matching closing bracket. If it's not empty, it means there are unclosed opening brackets, so it's invalid.

>[!example]- C++
>```cpp
>#include <stack>
>#include <unordered_map>
>#include <string>
>
>bool isValidParentheses(std::string s) {
>    std::stack<char> stack;
>    std::unordered_map<char, char> mapping = {
>        {')', '('},
>        {'}', '{'},
>        {']', '['}
>    };
>
>    for (char c : s) {
>        // If it's a closing bracket
>        if (mapping.find(c) != mapping.end()) {
>            if (stack.empty()) {
>                return false; // Closing bracket with no open one
>            }
>
>            char topElement = stack.top();
>            stack.pop();
>
>            if (mapping[c] != topElement) {
>                return false; // Mismatched brackets
>            }
>        } else { // It's an opening bracket
>            stack.push(c);
>        }
>    }
>
>    // Finally, the stack must be empty for the string to be valid
>    return stack.empty();
>}
>```

>[!example]- Java
>```java
>import java.util.Stack;
>import java.util.HashMap;
>
>public boolean isValidParentheses(String s) {
>    Stack<Character> stack = new Stack<>();
>    HashMap<Character, Character> mapping = new HashMap<>();
>    mapping.put(')', '(');
>    mapping.put('}', '{');
>    mapping.put(']', '[');
>
>    for (char c : s.toCharArray()) {
>        // If it's a closing bracket
>        if (mapping.containsKey(c)) {
>            if (stack.isEmpty()) {
>                return false; // Closing bracket with no open one
>            }
>
>            char topElement = stack.pop();
>
>            if (mapping.get(c) != topElement) {
>                return false; // Mismatched brackets
>            }
>        } else { // It's an opening bracket
>            stack.push(c);
>        }
>    }
>
>    // Finally, the stack must be empty for the string to be valid
>    return stack.isEmpty();
>}
>```

>[!example]- Python
>```python
>def is_valid_parentheses(s: str) -> bool:
>    stack = []
>    mapping = {")": "(", "}": "{", "]": "["}
>
>    for char in s:
>        if char in mapping: # It's a closing bracket
>            if not stack:
>                return False # Closing bracket with no open one
>
>            top_element = stack.pop()
>            if mapping[char] != top_element:
>                return False # Mismatched brackets
>        else: # It's an opening bracket
>            stack.append(char)
>
>    # Finally, the stack must be empty for the string to be valid
>    return not stack
>```

>[!example]- JavaScript
>```javascript
>function isValidParentheses(s) {
>    const stack = [];
>    const mapping = {
>        ')': '(',
>        '}': '{',
>        ']': '['
>    };
>
>    for (const char of s) {
>        // If it's a closing bracket
>        if (char in mapping) {
>            if (stack.length === 0) {
>                return false; // Closing bracket with no open one
>            }
>
>            const topElement = stack.pop();
>
>            if (mapping[char] !== topElement) {
>                return false; // Mismatched brackets
>            }
>        } else { // It's an opening bracket
>            stack.push(char);
>        }
>    }
>
>    // Finally, the stack must be empty for the string to be valid
>    return stack.length === 0;
>}
>```

### Other Common Problem Types
- **Simplifying Paths**: Used to handle directory structures (e.g., `..` means "go up," which is a pop operation).
- **Evaluating Expressions**: Converting infix expressions to postfix (Reverse Polish Notation) and evaluating them.
- **Backtracking/DFS Simulation**: An explicit stack can be used to implement DFS iteratively, which avoids recursion depth limits.
