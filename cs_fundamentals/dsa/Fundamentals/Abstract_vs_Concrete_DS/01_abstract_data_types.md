# Abstract Data Types

An Abstract Data Type (ADT) is a mathematical model for data types where a data type is defined by its behavior from the user's point of view, specifically in terms of possible values, possible operations, and the behavior of these operations.

## Core Concepts

### Abstraction

Abstraction shows only essential attributes and hides unnecessary implementation details. When you use a `Map` in your code, you don't need to know if it's using hashing, trees, or any other mechanism internally.

### Encapsulation

Encapsulation combines data and operations into a single unit, hiding internal complexity. Users interact through a defined interface without worrying about implementation.

## Common Abstract Data Types

### 1. Stack ADT

**Definition**: A collection with Last-In-First-Out (LIFO) access

**Operations**:
- `push(item)` - Add item to top
- `pop()` - Remove and return top item
- `peek()`/`top()` - View top item without removing
- `isEmpty()` - Check if stack is empty
- `size()` - Get number of elements

**Use Cases**: Function call management, undo operations, expression evaluation

### 2. Queue ADT

**Definition**: A collection with First-In-First-Out (FIFO) access

**Operations**:
- `enqueue(item)` - Add item to back
- `dequeue()` - Remove and return front item
- `peek()`/`front()` - View front item without removing
- `isEmpty()` - Check if queue is empty
- `size()` - Get number of elements

**Use Cases**: BFS, task scheduling, buffering

### 3. List ADT

**Definition**: An ordered collection of elements

**Operations**:
- `add(item)` / `add(index, item)` - Insert element
- `remove(index)` / `remove(item)` - Delete element
- `get(index)` - Access element
- `set(index, item)` - Modify element
- `size()` - Get length
- `indexOf(item)` - Find element position

### 4. Set ADT

**Definition**: A collection of unique elements

**Operations**:
- `add(item)` - Insert element (no duplicates)
- `remove(item)` - Delete element
- `contains(item)` - Check membership
- `size()` - Get number of elements
- `union(otherSet)` - Combine sets
- `intersection(otherSet)` - Common elements

### 5. Map/Dictionary ADT

**Definition**: A collection of key-value pairs

**Operations**:
- `put(key, value)` - Insert or update pair
- `get(key)` - Retrieve value by key
- `remove(key)` - Delete pair
- `containsKey(key)` - Check if key exists
- `keys()` - Get all keys
- `values()` - Get all values
- `size()` - Get number of pairs

### 6. Priority Queue ADT

**Definition**: A queue where elements have priorities

**Operations**:
- `insert(item, priority)` - Add with priority
- `extractMax()`/`extractMin()` - Remove highest/lowest priority
- `peek()` - View highest/lowest priority item
- `isEmpty()` - Check if empty
- `changePriority(item, newPriority)` - Update priority

### 7. Graph ADT

**Definition**: A set of vertices connected by edges

**Operations**:
- `addVertex(v)` - Add a vertex
- `addEdge(v1, v2)` - Add an edge
- `removeVertex(v)` - Remove a vertex
- `removeEdge(v1, v2)` - Remove an edge
- `getNeighbors(v)` - Get adjacent vertices
- `hasEdge(v1, v2)` - Check if edge exists

## ADT vs Implementation in Code

>[!example]- C++
>```cpp
>// ADT: Map
>// CDT: unordered_map (hash-based) or map (tree-based)
>std::unordered_map<std::string, int> wordCount;
>
>// ADT: List
>// CDT: vector (dynamic array) or list (linked list)
>std::vector<std::string> names;
>
>// ADT: Set
>// CDT: unordered_set (hash-based) or set (tree-based)
>std::unordered_set<int> uniqueNumbers;
>
>// ADT: Queue
>// CDT: queue (typically backed by deque)
>std::queue<std::string> tasks;
>
>// ADT: Stack
>// CDT: stack (typically backed by deque)
>std::stack<int> items;
>```

>[!example]- Java
>```java
>// ADT: Map
>// CDT: HashMap
>Map<String, Integer> wordCount = new HashMap<>();
>
>// ADT: List
>// CDT: ArrayList
>List<String> names = new ArrayList<>();
>
>// ADT: Set
>// CDT: HashSet
>Set<Integer> uniqueNumbers = new HashSet<>();
>
>// ADT: Queue
>// CDT: LinkedList (implements Queue interface)
>Queue<String> tasks = new LinkedList<>();
>
>// ADT: Stack
>// CDT: Stack class or ArrayDeque
>Stack<Integer> items = new Stack<>();
>```

>[!example]- Python
>```python
># Python abstracts this even more
># dict is both ADT and CDT
>word_count = {}  # HashMap implementation
>
># list is both ADT and CDT
>names = []  # Dynamic array implementation
>
># set is both ADT and CDT
>unique_numbers = set()  # Hash-based implementation
>
># collections.deque for Queue ADT
>from collections import deque
>tasks = deque()
>
># List can be used as Stack ADT
>items = []  # Use append() and pop()
>```

>[!example]- JavaScript
>```javascript
>// ADT: Map
>// CDT: Map or plain object
>const wordCount = new Map();
>// Or: const wordCount = {};
>
>// ADT: List
>// CDT: Array
>const names = [];
>
>// ADT: Set
>// CDT: Set
>const uniqueNumbers = new Set();
>
>// ADT: Queue
>// CDT: Array (use push/shift or unshift/pop)
>const tasks = [];
>
>// ADT: Stack
>// CDT: Array (use push/pop)
>const items = [];
>```

## Benefits of ADT Thinking

1. **Separation of Concerns**: Focus on what you need, not how it's done
2. **Flexibility**: Easy to swap implementations
3. **Abstraction**: Hide complex details
4. **Reusability**: Define once, implement many ways
5. **Clarity**: Clear contracts and expectations
