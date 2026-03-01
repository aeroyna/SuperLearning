## Counting with Hash Maps

One of the most frequent and fundamental use cases for a hash map in coding interviews is to count the occurrences of items in a collection. This pattern is simple, powerful, and serves as a building block for many more complex problems.

### Core Idea
The pattern is straightforward:
- **Keys**: The items in the collection you want to count (e.g., numbers in an array, characters in a string).
- **Values**: The frequency (the count) of each item.

You iterate through the collection. For each item, you check if it's already a key in your hash map.
- If it is, you increment its corresponding value (the count).
- If it's not, you add it to the map with a starting count of 1.

### Implementation Methods

#### Method 1: Manual Check
This is the most fundamental way, which works in any language.

>[!example]- C++
>```cpp
>#include <unordered_map>
>#include <string>
>#include <iostream>
>
>int main() {
>    std::string s = "hello world";
>    std::unordered_map<char, int> freq_map;
>
>    for (char c : s) {
>        if (freq_map.find(c) != freq_map.end()) {
>            freq_map[c] += 1;
>        } else {
>            freq_map[c] = 1;
>        }
>    }
>
>    // freq_map will be {'h': 1, 'e': 1, 'l': 3, 'o': 2, ' ': 1, 'w': 1, 'r': 1, 'd': 1}
>    return 0;
>}
>```

>[!example]- Java
>```java
>import java.util.HashMap;
>import java.util.Map;
>
>public class CountingExample {
>    public static void main(String[] args) {
>        String s = "hello world";
>        Map<Character, Integer> freqMap = new HashMap<>();
>
>        for (char c : s.toCharArray()) {
>            if (freqMap.containsKey(c)) {
>                freqMap.put(c, freqMap.get(c) + 1);
>            } else {
>                freqMap.put(c, 1);
>            }
>        }
>
>        // freqMap will be {'h': 1, 'e': 1, 'l': 3, 'o': 2, ' ': 1, 'w': 1, 'r': 1, 'd': 1}
>    }
>}
>```

>[!example]- Python
>```python
>s = "hello world"
>freq_map = {}
>
>for char in s:
>    if char in freq_map:
>        freq_map[char] += 1
>    else:
>        freq_map[char] = 1
>
># freq_map will be {'h': 1, 'e': 1, 'l': 3, 'o': 2, ' ': 1, 'w': 1, 'r': 1, 'd': 1}
>```

>[!example]- JavaScript
>```javascript
>let s = "hello world";
>let freqMap = {};
>
>for (let char of s) {
>    if (char in freqMap) {
>        freqMap[char] += 1;
>    } else {
>        freqMap[char] = 1;
>    }
>}
>
>// freqMap will be {'h': 1, 'e': 1, 'l': 3, 'o': 2, ' ': 1, 'w': 1, 'r': 1, 'd': 1}
>```

#### Method 2: Using Default Values
This is a slightly more concise version of the manual check. Each language provides methods to get values with defaults.

>[!example]- C++
>```cpp
>#include <unordered_map>
>#include <string>
>
>int main() {
>    std::string s = "hello world";
>    std::unordered_map<char, int> freq_map;
>
>    for (char c : s) {
>        // Using [] operator which default-initializes to 0 for int
>        freq_map[c] += 1;
>    }
>
>    // Or explicitly check and insert with default
>    for (char c : s) {
>        auto it = freq_map.find(c);
>        if (it != freq_map.end()) {
>            it->second += 1;
>        } else {
>            freq_map[c] = 1;
>        }
>    }
>
>    return 0;
>}
>```

>[!example]- Java
>```java
>import java.util.HashMap;
>import java.util.Map;
>
>public class CountingExample {
>    public static void main(String[] args) {
>        String s = "hello world";
>        Map<Character, Integer> freqMap = new HashMap<>();
>
>        for (char c : s.toCharArray()) {
>            freqMap.put(c, freqMap.getOrDefault(c, 0) + 1);
>        }
>    }
>}
>```

>[!example]- Python
>```python
>s = "hello world"
>freq_map = {}
>
>for char in s:
>    freq_map[char] = freq_map.get(char, 0) + 1
>```

>[!example]- JavaScript
>```javascript
>let s = "hello world";
>let freqMap = {};
>
>for (let char of s) {
>    freqMap[char] = (freqMap[char] || 0) + 1;
>}
>
>// Alternative using Map
>let freqMapAlt = new Map();
>for (let char of s) {
>    freqMapAlt.set(char, (freqMapAlt.get(char) || 0) + 1);
>}
>```

#### Method 3: Using Language-Specific Helpers
Many languages provide convenient data structures that automatically handle default values.

>[!example]- C++
>```cpp
>#include <unordered_map>
>#include <string>
>
>int main() {
>    std::string s = "hello world";
>    std::unordered_map<char, int> freq_map;
>
>    // In C++, the [] operator automatically default-initializes
>    // new entries to 0 for integer types
>    for (char c : s) {
>        freq_map[c]++;
>    }
>
>    return 0;
>}
>```

>[!example]- Java
>```java
>import java.util.HashMap;
>import java.util.Map;
>
>public class CountingExample {
>    public static void main(String[] args) {
>        String s = "hello world";
>        Map<Character, Integer> freqMap = new HashMap<>();
>
>        // Java requires explicit handling or use of compute methods
>        for (char c : s.toCharArray()) {
>            freqMap.compute(c, (key, val) -> val == null ? 1 : val + 1);
>        }
>
>        // Or using merge (Java 8+)
>        for (char c : s.toCharArray()) {
>            freqMap.merge(c, 1, Integer::sum);
>        }
>    }
>}
>```

>[!example]- Python
>```python
>from collections import defaultdict
>
>s = "hello world"
># A defaultdict(int) will create items with a default value of 0
>freq_map = defaultdict(int)
>
>for char in s:
>    freq_map[char] += 1
>```

>[!example]- JavaScript
>```javascript
>let s = "hello world";
>let freqMap = new Map();
>
>for (let char of s) {
>    freqMap.set(char, (freqMap.get(char) || 0) + 1);
>}
>```

#### Method 4: Using Built-in Counter Utilities
Some languages provide specialized counting utilities.

>[!example]- C++
>```cpp
>#include <string>
>#include <map>
>#include <algorithm>
>
>int main() {
>    std::string s = "hello world";
>
>    // C++ doesn't have a built-in Counter, but we can use map
>    std::map<char, int> freq_map;
>    for (char c : s) {
>        freq_map[c]++;
>    }
>
>    // Or use count algorithm for specific elements
>    int count_l = std::count(s.begin(), s.end(), 'l'); // Returns 3
>
>    return 0;
>}
>```

>[!example]- Java
>```java
>import java.util.HashMap;
>import java.util.Map;
>
>public class CountingExample {
>    public static void main(String[] args) {
>        String s = "hello world";
>        Map<Character, Integer> freqMap = new HashMap<>();
>
>        // Java doesn't have a built-in Counter class
>        // Use standard map operations
>        for (char c : s.toCharArray()) {
>            freqMap.merge(c, 1, Integer::sum);
>        }
>
>        // Or use streams (Java 8+)
>        Map<Character, Long> freqMapStream = s.chars()
>            .mapToObj(c -> (char) c)
>            .collect(java.util.stream.Collectors.groupingBy(
>                c -> c,
>                java.util.stream.Collectors.counting()
>            ));
>    }
>}
>```

>[!example]- Python
>```python
>from collections import Counter
>
>s = "hello world"
>freq_map = Counter(s)
>```

>[!example]- JavaScript
>```javascript
>// JavaScript doesn't have a built-in Counter
>// Use reduce for a functional approach
>let s = "hello world";
>
>let freqMap = s.split('').reduce((map, char) => {
>    map[char] = (map[char] || 0) + 1;
>    return map;
>}, {});
>
>// Or using Map with reduce
>let freqMapAlt = Array.from(s).reduce((map, char) => {
>    map.set(char, (map.get(char) || 0) + 1);
>    return map;
>}, new Map());
>```

### Example Application: Majority Element (LeetCode #169)
**Problem**: Given an array `nums` of size `n`, return the majority element. The majority element is the element that appears more than `⌊n / 2⌋` times.

**Solution**: This is a perfect use case for a frequency map.
1. Build a hash map to count the frequency of each number in `nums`.
2. Iterate through the hash map.
3. The first element you find with a count greater than `n / 2` is the answer.

>[!example]- C++
>```cpp
>#include <vector>
>#include <unordered_map>
>
>int majorityElement(std::vector<int>& nums) {
>    std::unordered_map<int, int> counts;
>    for (int num : nums) {
>        counts[num]++;
>    }
>
>    int n = nums.size();
>    for (const auto& [num, count] : counts) {
>        if (count > n / 2) {
>            return num;
>        }
>    }
>    return -1; // Should not be reached given the problem constraints
>}
>```

>[!example]- Java
>```java
>import java.util.HashMap;
>import java.util.Map;
>
>public class Solution {
>    public int majorityElement(int[] nums) {
>        Map<Integer, Integer> counts = new HashMap<>();
>        for (int num : nums) {
>            counts.put(num, counts.getOrDefault(num, 0) + 1);
>        }
>
>        int n = nums.length;
>        for (Map.Entry<Integer, Integer> entry : counts.entrySet()) {
>            if (entry.getValue() > n / 2) {
>                return entry.getKey();
>            }
>        }
>        return -1; // Should not be reached given the problem constraints
>    }
>}
>```

>[!example]- Python
>```python
>def majority_element(nums):
>    counts = {}
>    for num in nums:
>        counts[num] = counts.get(num, 0) + 1
>
>    n = len(nums)
>    for num, count in counts.items():
>        if count > n / 2:
>            return num
>    return -1 # Should not be reached given the problem constraints
>```

>[!example]- JavaScript
>```javascript
>function majorityElement(nums) {
>    const counts = new Map();
>    for (const num of nums) {
>        counts.set(num, (counts.get(num) || 0) + 1);
>    }
>
>    const n = nums.length;
>    for (const [num, count] of counts) {
>        if (count > n / 2) {
>            return num;
>        }
>    }
>    return -1; // Should not be reached given the problem constraints
>}
>```

While this problem has a clever O(1) space solution (Boyer-Moore Voting Algorithm), the hash map approach is often the most intuitive and is a perfectly acceptable O(n) time and O(n) space solution in an interview.
