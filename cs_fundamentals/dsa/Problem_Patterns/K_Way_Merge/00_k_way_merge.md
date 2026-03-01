# K-way Merge

The K-way Merge pattern is used to merge `K` sorted data structures (arrays, lists, matrices) into a single sorted structure. It is a generalization of the standard 2-way merge sort step.

## Concept

When we have `K` sorted lists, finding the smallest element among all lists requires checking the current head of each list.
- A brute force approach checks all `K` heads in `O(K)` time per element.
- Using a **Min-Heap**, we can find the smallest element in **O(1)** and remove/replace it in **O(log K)**.

## Algorithm

1.  Insert the first element of each of the `K` lists into a **Min-Heap**. The heap stores tuples like `(value, list_index, element_index)`.
2.  While the heap is not empty:
    -   Extract the minimum element `(val, list_idx, elem_idx)` from the heap and add it to the result.
    -   If the list at `list_idx` has a next element, insert that next element into the heap.

## Implementation (Merge K Sorted Arrays)

>[!example]- C++
>```cpp
>struct Element {
>    int val;
>    int listIdx;
>    int nextElemIdx;
>    
>    bool operator>(const Element& other) const {
>        return val > other.val;
>    }
>};
>
>vector<int> mergeKLists(vector<vector<int>>& lists) {
>    priority_queue<Element, vector<Element>, greater<Element>> minHeap;
>    vector<int> result;
>    
>    // Push first element of each list
>    for (int i = 0; i < lists.size(); i++) {
>        if (!lists[i].empty()) {
>            minHeap.push({lists[i][0], i, 1});
>        }
>    }
>    
>    while (!minHeap.empty()) {
>        Element curr = minHeap.top();
>        minHeap.pop();
>        result.push_back(curr.val);
>        
>        if (curr.nextElemIdx < lists[curr.listIdx].size()) {
>            minHeap.push({lists[curr.listIdx][curr.nextElemIdx], curr.listIdx, curr.nextElemIdx + 1});
>        }
>    }
>    return result;
>}
>```

>[!example]- Java
>```java
>public List<Integer> mergeKLists(List<List<Integer>> lists) {
>    PriorityQueue<int[]> minHeap = new PriorityQueue<>((a, b) -> a[0] - b[0]);
>    List<Integer> result = new ArrayList<>();
>    
>    // Push first element of each list: {val, listIdx, nextElemIdx}
>    for (int i = 0; i < lists.size(); i++) {
>        if (!lists.get(i).isEmpty()) {
>            minHeap.offer(new int[]{lists.get(i).get(0), i, 1});
>        }
>    }
>    
>    while (!minHeap.isEmpty()) {
>        int[] curr = minHeap.poll();
>        result.add(curr[0]);
>        int listIdx = curr[1];
>        int nextElemIdx = curr[2];
>        
>        if (nextElemIdx < lists.get(listIdx).size()) {
>            minHeap.offer(new int[]{lists.get(listIdx).get(nextElemIdx), listIdx, nextElemIdx + 1});
>        }
>    }
>    return result;
>}
>```

>[!example]- Python
>```python
>import heapq
>
>def merge_k_lists(lists):
>    min_heap = []
>    result = []
>    
>    # Push first element of each list
>    # Heap stores: (value, list_idx, next_elem_idx)
>    for i, lst in enumerate(lists):
>        if lst:
>            heapq.heappush(min_heap, (lst[0], i, 1))
>            
>    while min_heap:
>        val, list_idx, next_elem_idx = heapq.heappop(min_heap)
>        result.append(val)
>        
>        if next_elem_idx < len(lists[list_idx]):
>            next_val = lists[list_idx][next_elem_idx]
>            heapq.heappush(min_heap, (next_val, list_idx, next_elem_idx + 1))
>            
>    return result
>```

>[!example]- JavaScript
>```javascript
>function mergeKLists(lists) {
>    // Note: JavaScript doesn't have a built-in PriorityQueue.
>    // Using a simplified array-based approach for demonstration (not O(log K) insert/pop).
>    // In a real interview, you'd implement a MinHeap class or assume one exists.
>    
>    const minHeap = new MinHeap(); // Assuming MinHeap class exists
>    const result = [];
>    
>    for (let i = 0; i < lists.length; i++) {
>        if (lists[i].length > 0) {
>            // { val, listIdx, nextElemIdx }
>            minHeap.push({ val: lists[i][0], listIdx: i, nextElemIdx: 1 });
>        }
>    }
>    
>    while (!minHeap.isEmpty()) {
>        const curr = minHeap.pop();
>        result.push(curr.val);
>        
>        if (curr.nextElemIdx < lists[curr.listIdx].length) {
>            minHeap.push({ 
>                val: lists[curr.listIdx][curr.nextElemIdx], 
>                listIdx: curr.listIdx, 
>                nextElemIdx: curr.nextElemIdx + 1 
>            });
>        }
>    }
>    return result;
>}
>```

## Pattern Recognition

**Signals**:
- "Merge K sorted lists"
- "Find Kth smallest number in M sorted lists"
- "Find smallest range covering elements from K lists"
- Input consists of multiple sorted sequences.

## Common Problems

### 1. Merge K Sorted Lists
Merge `k` linked lists and return it as one sorted list.
- **Approach**: Maintain a Min-Heap of size `k`. Push head of each list. Pop min, attach to result, push next node from that list.

### 2. Kth Smallest Number in M Sorted Lists
Given `M` sorted lists, find the Kth smallest element in the global sorted order.
- **Approach**: Same as above. Perform `k` pop operations. The Kth popped element is the answer.

### 3. Smallest Range Covering Elements from K Lists
Find the smallest range `[a, b]` that includes at least one number from each of the `k` lists.
- **Approach**: Keep a Min-Heap of size `k` (one from each list). Track the current `max` element in the heap separately. Range is `current_max - heap.min()`. Pop min, push next. Update range if smaller.

### 4. Find K Pairs with Smallest Sums
Given two sorted arrays, find `k` pairs `(u, v)` with the smallest sums.
- **Approach**: Treat this as merging `N` sorted lists (where row `i` is `nums1[i] + nums2[0...M]`).

## Complexity Analysis

- **Time Complexity**: **O(N log K)**, where `N` is the total number of elements across all lists and `K` is the number of lists.
    - Heap operations take `O(log K)`. We perform `N` such operations.
- **Space Complexity**: **O(K)** to store the elements in the heap.


### Practice
- [Practice Problems](Practice_Problems/00_practice_problems.md)