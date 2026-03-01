## Hash Set

A hash set is a data structure used to store a collection of **unique** elements. It is extremely efficient for checking if an element exists in the set.

### Core Idea
A hash set is built on top of a hash table. When you add an element, the hash set calculates a hash value for the element and stores it at the corresponding location. Because of this, adding, removing, and checking for existence are all O(1) operations on average.

The key feature is that hash sets do not allow duplicate elements. If you try to add an element that is already in the set, the operation is simply ignored.

### Use Cases
- **Checking for duplicates**: The most common use case. You can iterate through a collection and add elements to a hash set to quickly identify duplicates.
- **Finding unique elements**: Simply add all elements from a collection into a hash set; the result is a set of the unique elements.
- **Set operations**: Performing mathematical set operations like union, intersection, and difference.

### Implementation Examples

>[!example]- C++
>```cpp
>#include <unordered_set>
>#include <string>
>#include <iostream>
>
>int main() {
>    // 1. Initialize a hash set
>    std::unordered_set<std::string> visited_nodes;
>
>    // 2. Add elements
>    visited_nodes.insert("A");
>    visited_nodes.insert("B");
>    visited_nodes.insert("C");
>    visited_nodes.insert("B"); // This is ignored, 'B' is already present
>
>    // visited_nodes is now {"A", "B", "C"}
>
>    // 3. Check for existence (O(1) on average)
>    if (visited_nodes.find("C") != visited_nodes.end()) {
>        std::cout << "Node 'C' has been visited." << std::endl;
>    }
>
>    // 4. Remove an element
>    visited_nodes.erase("A"); // Returns the number of elements removed (0 or 1)
>
>    // Check before removing (safer)
>    if (visited_nodes.find("Z") != visited_nodes.end()) {
>        visited_nodes.erase("Z");
>    }
>
>    // 5. Get the size
>    int num_unique_nodes = visited_nodes.size(); // 2
>
>    // 6. Iterate over the set (order is not guaranteed)
>    for (const std::string& node : visited_nodes) {
>        std::cout << node << std::endl;
>    }
>
>    return 0;
>}
>```

>[!example]- Java
>```java
>import java.util.HashSet;
>import java.util.Set;
>
>public class HashSetExample {
>    public static void main(String[] args) {
>        // 1. Initialize a hash set
>        Set<String> visitedNodes = new HashSet<>();
>
>        // 2. Add elements
>        visitedNodes.add("A");
>        visitedNodes.add("B");
>        visitedNodes.add("C");
>        visitedNodes.add("B"); // This is ignored, 'B' is already present
>
>        // visitedNodes is now {"A", "B", "C"}
>
>        // 3. Check for existence (O(1) on average)
>        if (visitedNodes.contains("C")) {
>            System.out.println("Node 'C' has been visited.");
>        }
>
>        // 4. Remove an element
>        visitedNodes.remove("A"); // Returns true if element was present
>
>        // Safe removal (no exception if not found)
>        visitedNodes.remove("Z"); // Returns false if 'Z' is not found
>
>        // 5. Get the size
>        int numUniqueNodes = visitedNodes.size(); // 2
>
>        // 6. Iterate over the set (order is not guaranteed)
>        for (String node : visitedNodes) {
>            System.out.println(node);
>        }
>    }
>}
>```

>[!example]- Python
>```python
># 1. Initialize a hash set
>visited_nodes = set()
>
># 2. Add elements
>visited_nodes.add('A')
>visited_nodes.add('B')
>visited_nodes.add('C')
>visited_nodes.add('B') # This is ignored, 'B' is already present
>
># visited_nodes is now {'A', 'B', 'C'}
>
># 3. Check for existence (O(1) on average)
>if 'C' in visited_nodes:
>    print("Node 'C' has been visited.")
>
># 4. Remove an element
>visited_nodes.remove('A') # Raises a KeyError if 'A' is not found
>visited_nodes.discard('Z') # Does nothing if 'Z' is not found (safer)
>
># 5. Get the size
>num_unique_nodes = len(visited_nodes) # 2
>
># 6. Iterate over the set (order is not guaranteed)
>for node in visited_nodes:
>    print(node)
>```

>[!example]- JavaScript
>```javascript
>// 1. Initialize a hash set
>let visitedNodes = new Set();
>
>// 2. Add elements
>visitedNodes.add('A');
>visitedNodes.add('B');
>visitedNodes.add('C');
>visitedNodes.add('B'); // This is ignored, 'B' is already present
>
>// visitedNodes is now Set(3) {'A', 'B', 'C'}
>
>// 3. Check for existence (O(1) on average)
>if (visitedNodes.has('C')) {
>    console.log("Node 'C' has been visited.");
>}
>
>// 4. Remove an element
>visitedNodes.delete('A'); // Returns true if element was present
>visitedNodes.delete('Z'); // Returns false if 'Z' is not found (safe)
>
>// 5. Get the size
>let numUniqueNodes = visitedNodes.size; // 2
>
>// 6. Iterate over the set (maintains insertion order in JavaScript)
>for (let node of visitedNodes) {
>    console.log(node);
>}
>
>// Alternative iteration using forEach
>visitedNodes.forEach(node => {
>    console.log(node);
>});
>```
