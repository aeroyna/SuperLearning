# std::queue

`std::queue` is a container adapter that provides FIFO (First-In-First-Out) queue operations.

## Include Header

```cpp
#include <queue>
```

## Creating a Queue

```cpp
std::queue<int> q1;                      // Empty queue (uses deque by default)
std::queue<int, std::list<int>> q2;      // Queue backed by list
```

## Key Operations

| Operation | Description | Complexity |
|-----------|-------------|------------|
| `push(x)` | Add element to back | O(1) |
| `pop()` | Remove front element | O(1) |
| `front()` | Access front element | O(1) |
| `back()` | Access back element | O(1) |
| `empty()` | Check if empty | O(1) |
| `size()` | Get number of elements | O(1) |

```cpp
std::queue<int> q;

q.push(10);
q.push(20);
q.push(30);

std::cout << q.front();  // 10 (first pushed)
std::cout << q.back();   // 30 (last pushed)

q.pop();                 // Remove 10
std::cout << q.front();  // 20
```

## Queue vs Stack

| Aspect | queue | stack |
|--------|-------|-------|
| Order | FIFO | LIFO |
| Access front | `front()` | - |
| Access back/top | `back()` | `top()` |
| Remove from | front | top |

## Common Use Cases

### 1. BFS (Breadth-First Search)

```cpp
void bfs(int start, const std::vector<std::vector<int>>& graph) {
    std::queue<int> q;
    std::vector<bool> visited(graph.size(), false);

    q.push(start);
    visited[start] = true;

    while (!q.empty()) {
        int node = q.front();
        q.pop();

        std::cout << "Visiting: " << node << "\n";

        for (int neighbor : graph[node]) {
            if (!visited[neighbor]) {
                visited[neighbor] = true;
                q.push(neighbor);
            }
        }
    }
}
```

### 2. Task Scheduling

```cpp
struct Task {
    std::string name;
    int priority;
};

std::queue<Task> taskQueue;
taskQueue.push({"Task A", 1});
taskQueue.push({"Task B", 2});

while (!taskQueue.empty()) {
    Task t = taskQueue.front();
    taskQueue.pop();
    std::cout << "Processing: " << t.name << "\n";
}
```

### 3. Producer-Consumer Pattern

```cpp
std::queue<int> buffer;
std::mutex mtx;

void producer() {
    for (int i = 0; i < 10; ++i) {
        std::lock_guard<std::mutex> lock(mtx);
        buffer.push(i);
    }
}

void consumer() {
    while (true) {
        std::lock_guard<std::mutex> lock(mtx);
        if (!buffer.empty()) {
            int item = buffer.front();
            buffer.pop();
            // process item
        }
    }
}
```

## Key Takeaways

- FIFO data structure
- Container adapter (wraps deque or list)
- Access both front and back
- No iterators
- Use for: BFS, scheduling, buffering

## Common Interview Questions

> [!question]- How would you implement a queue using two stacks?
> Push to stack1. To pop, if stack2 is empty, transfer all from stack1 to stack2 (reversing order), then pop from stack2.

> [!question]- Why can't queue use vector as underlying container?
> Queue needs efficient front removal. Vector's `pop_front` is O(n). Deque and list both have O(1) front removal.
