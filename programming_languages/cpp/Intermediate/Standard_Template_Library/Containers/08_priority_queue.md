# std::priority_queue

`std::priority_queue` is a container adapter that provides constant-time access to the **largest** (by default) element.

## Include Header

```cpp
#include <queue>  // Same header as queue
```

## Creating a Priority Queue

```cpp
// Max heap (default) - largest element at top
std::priority_queue<int> maxPQ;

// Min heap - smallest element at top
std::priority_queue<int, std::vector<int>, std::greater<int>> minPQ;

// From vector
std::vector<int> v = {3, 1, 4, 1, 5};
std::priority_queue<int> pq(v.begin(), v.end());

// Custom comparator
auto cmp = [](int a, int b) { return a > b; };  // Min heap
std::priority_queue<int, std::vector<int>, decltype(cmp)> customPQ(cmp);
```

## Key Operations

| Operation | Description | Complexity |
|-----------|-------------|------------|
| `push(x)` | Add element | O(log n) |
| `pop()` | Remove top element | O(log n) |
| `top()` | Access top element | O(1) |
| `empty()` | Check if empty | O(1) |
| `size()` | Get number of elements | O(1) |

```cpp
std::priority_queue<int> pq;

pq.push(30);
pq.push(10);
pq.push(50);
pq.push(20);

std::cout << pq.top();  // 50 (largest)

pq.pop();
std::cout << pq.top();  // 30

// Extract all in sorted order (descending)
while (!pq.empty()) {
    std::cout << pq.top() << " ";  // 30 20 10
    pq.pop();
}
```

## Min Heap

```cpp
// Method 1: Using std::greater
std::priority_queue<int, std::vector<int>, std::greater<int>> minPQ;
minPQ.push(30);
minPQ.push(10);
minPQ.push(50);
std::cout << minPQ.top();  // 10 (smallest)

// Method 2: Negate values (for simple cases)
std::priority_queue<int> pq;
pq.push(-30);
pq.push(-10);
pq.push(-50);
std::cout << -pq.top();  // 10 (smallest)
```

## Custom Comparator

```cpp
struct Task {
    std::string name;
    int priority;
};

// Custom comparator
struct TaskCompare {
    bool operator()(const Task& a, const Task& b) {
        return a.priority < b.priority;  // Higher priority first
    }
};

std::priority_queue<Task, std::vector<Task>, TaskCompare> taskQueue;
taskQueue.push({"Low", 1});
taskQueue.push({"High", 10});
taskQueue.push({"Medium", 5});

std::cout << taskQueue.top().name;  // "High"
```

## Common Use Cases

### 1. K Largest Elements

```cpp
std::vector<int> kLargest(const std::vector<int>& nums, int k) {
    // Min heap of size k
    std::priority_queue<int, std::vector<int>, std::greater<int>> minPQ;

    for (int num : nums) {
        minPQ.push(num);
        if (minPQ.size() > k) {
            minPQ.pop();  // Remove smallest
        }
    }

    std::vector<int> result;
    while (!minPQ.empty()) {
        result.push_back(minPQ.top());
        minPQ.pop();
    }
    return result;
}
```

### 2. Dijkstra's Algorithm

```cpp
void dijkstra(int start, const std::vector<std::vector<std::pair<int,int>>>& graph) {
    std::vector<int> dist(graph.size(), INT_MAX);
    // Min heap: {distance, node}
    std::priority_queue<std::pair<int,int>,
                       std::vector<std::pair<int,int>>,
                       std::greater<std::pair<int,int>>> pq;

    dist[start] = 0;
    pq.push({0, start});

    while (!pq.empty()) {
        auto [d, u] = pq.top();
        pq.pop();

        if (d > dist[u]) continue;

        for (auto [v, w] : graph[u]) {
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                pq.push({dist[v], v});
            }
        }
    }
}
```

### 3. Merge K Sorted Lists

```cpp
ListNode* mergeKLists(std::vector<ListNode*>& lists) {
    auto cmp = [](ListNode* a, ListNode* b) { return a->val > b->val; };
    std::priority_queue<ListNode*, std::vector<ListNode*>, decltype(cmp)> pq(cmp);

    for (auto list : lists) {
        if (list) pq.push(list);
    }

    ListNode dummy(0);
    ListNode* tail = &dummy;

    while (!pq.empty()) {
        ListNode* node = pq.top();
        pq.pop();
        tail->next = node;
        tail = tail->next;
        if (node->next) pq.push(node->next);
    }

    return dummy.next;
}
```

## Key Takeaways

- Heap-based data structure
- Max heap by default (largest on top)
- Use `std::greater<T>` for min heap
- O(log n) push/pop, O(1) top
- No random access or iteration
- Common in: shortest path, scheduling, k-th element problems

## Common Interview Questions

> [!question]- How do you create a min heap?
> Use `std::priority_queue<int, std::vector<int>, std::greater<int>>` or use a custom comparator.

> [!question]- What's the time complexity to build a priority queue from n elements?
> O(n) when using iterator constructor, O(n log n) when pushing one by one.

> [!question]- Can you iterate through a priority_queue?
> No. You can only access the top. To iterate, you must pop elements (destructive).
