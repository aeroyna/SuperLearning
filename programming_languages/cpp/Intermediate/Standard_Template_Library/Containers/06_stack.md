# std::stack

`std::stack` is a container adapter that provides LIFO (Last-In-First-Out) stack operations.

## Include Header

```cpp
#include <stack>
```

## Creating a Stack

```cpp
std::stack<int> s1;                        // Empty stack (uses deque by default)
std::stack<int, std::vector<int>> s2;      // Stack backed by vector
std::stack<int, std::list<int>> s3;        // Stack backed by list
```

## Key Operations

| Operation | Description | Complexity |
|-----------|-------------|------------|
| `push(x)` | Add element to top | O(1) |
| `pop()` | Remove top element | O(1) |
| `top()` | Access top element | O(1) |
| `empty()` | Check if empty | O(1) |
| `size()` | Get number of elements | O(1) |

```cpp
std::stack<int> s;

s.push(10);
s.push(20);
s.push(30);

std::cout << s.top();  // 30 (last pushed)

s.pop();               // Remove 30
std::cout << s.top();  // 20

std::cout << s.size(); // 2
std::cout << s.empty(); // false
```

## Important Notes

- `pop()` does NOT return the removed element (use `top()` first)
- No iterators - can only access top
- No `clear()` method - must pop until empty

```cpp
// Get and remove top element
int value = s.top();  // Get value first
s.pop();              // Then remove
```

## Common Use Cases

### 1. Balanced Parentheses

```cpp
bool isBalanced(const std::string& expr) {
    std::stack<char> s;

    for (char c : expr) {
        if (c == '(' || c == '[' || c == '{') {
            s.push(c);
        } else if (c == ')' || c == ']' || c == '}') {
            if (s.empty()) return false;
            char top = s.top();
            s.pop();
            if ((c == ')' && top != '(') ||
                (c == ']' && top != '[') ||
                (c == '}' && top != '{')) {
                return false;
            }
        }
    }
    return s.empty();
}
```

### 2. Reverse a Sequence

```cpp
void reverseString(std::string& str) {
    std::stack<char> s;
    for (char c : str) s.push(c);

    for (size_t i = 0; i < str.size(); ++i) {
        str[i] = s.top();
        s.pop();
    }
}
```

### 3. Function Call Simulation

```cpp
std::stack<std::string> callStack;
callStack.push("main()");
callStack.push("foo()");
callStack.push("bar()");

while (!callStack.empty()) {
    std::cout << "Returning from: " << callStack.top() << "\n";
    callStack.pop();
}
```

## Key Takeaways

- LIFO data structure
- Container adapter (wraps deque, vector, or list)
- Only top element accessible
- No iterators
- Use for: parsing, backtracking, undo operations

## Common Interview Questions

> [!question]- Why doesn't pop() return the removed value?
> Exception safety. If the copy constructor throws after removing the element, the element is lost. Separating access (top) and removal (pop) is safer.

> [!question]- What's the default underlying container?
> `std::deque`. You can also use `vector` or `list`.
