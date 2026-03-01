## Designing Custom Hash Keys

While hash tables are powerful, their effectiveness depends on choosing the right key. Sometimes, the raw data isn't suitable to be a key directly. The art of using a hash map often lies in designing a **canonical representation** for a group of items to serve as the key.

### Core Idea
The goal is to devise a consistent way to represent objects so that all objects that should be considered "the same" map to the exact same key. This key should be an **immutable** type (like a string, a number, or a tuple) to be hashable.

This process guarantees that:
1.  All items belonging to the same logical group produce the same key.
2.  Items belonging to different groups produce different keys.

### Classic Example: Group Anagrams (LeetCode #49)
This is the quintessential key design problem.

**Problem**: Given an array of strings, group the anagrams together.
`Input: ["eat", "tea", "tan", "ate", "nat", "bat"]`
`Output: [["bat"], ["nat", "tan"], ["ate", "eat", "tea"]]`

**Insight**: Two strings are anagrams if and only if they are composed of the same characters with the same frequencies.

**Key Design**: How can we create a single, canonical representation for all anagrams?
-   **Method 1: Sorting**. The sorted version of a string is a perfect canonical key. `"eat"`, `"tea"`, and `"ate"` all become `"aet"` when sorted.
-   **Method 2: Character Counts**. For strings with only lowercase English letters, we can use a tuple of 26 integers, where each integer represents the count of a character. For `"eat"`, the key would be `(1, 0, 0, 1, 1, 0, ..., 1, ...)` where the 1s correspond to the counts of `a`, `e`, and `t`. This is also a valid, immutable key.

#### Implementation (Sorting Method)

>[!example]- C++
>```cpp
>#include <vector>
>#include <string>
>#include <unordered_map>
>#include <algorithm>
>
>std::vector<std::vector<std::string>> groupAnagrams(std::vector<std::string>& strs) {
>    // The hash map will map the canonical key (sorted string)
>    // to a list of its anagrams.
>    std::unordered_map<std::string, std::vector<std::string>> anagram_map;
>
>    for (const std::string& s : strs) {
>        // Create the canonical key
>        std::string key = s;
>        std::sort(key.begin(), key.end());
>
>        // Group the original string under this key
>        anagram_map[key].push_back(s);
>    }
>
>    // The values of the hash map are the grouped anagrams
>    std::vector<std::vector<std::string>> result;
>    for (const auto& [key, group] : anagram_map) {
>        result.push_back(group);
>    }
>
>    return result;
>}
>```

>[!example]- Java
>```java
>import java.util.*;
>
>public class Solution {
>    public List<List<String>> groupAnagrams(String[] strs) {
>        // The hash map will map the canonical key (sorted string)
>        // to a list of its anagrams.
>        Map<String, List<String>> anagramMap = new HashMap<>();
>
>        for (String s : strs) {
>            // Create the canonical key
>            char[] chars = s.toCharArray();
>            Arrays.sort(chars);
>            String key = new String(chars);
>
>            // Group the original string under this key
>            anagramMap.putIfAbsent(key, new ArrayList<>());
>            anagramMap.get(key).add(s);
>        }
>
>        // The values of the hash map are the grouped anagrams
>        return new ArrayList<>(anagramMap.values());
>    }
>}
>```

>[!example]- Python
>```python
>from collections import defaultdict
>
>def group_anagrams(strs):
>    # The hash map will map the canonical key (sorted string)
>    # to a list of its anagrams.
>    anagram_map = defaultdict(list)
>
>    for s in strs:
>        # Create the canonical key
>        key = "".join(sorted(s))
>
>        # Group the original string under this key
>        anagram_map[key].append(s)
>
>    # The values of the hash map are the grouped anagrams
>    return list(anagram_map.values())
>```

>[!example]- JavaScript
>```javascript
>function groupAnagrams(strs) {
>    // The hash map will map the canonical key (sorted string)
>    // to a list of its anagrams.
>    const anagramMap = new Map();
>
>    for (const s of strs) {
>        // Create the canonical key
>        const key = s.split('').sort().join('');
>
>        // Group the original string under this key
>        if (!anagramMap.has(key)) {
>            anagramMap.set(key, []);
>        }
>        anagramMap.get(key).push(s);
>    }
>
>    // The values of the hash map are the grouped anagrams
>    return Array.from(anagramMap.values());
>}
>```

### Other Key Design Strategies

- **Using Tuples for Coordinates**: When dealing with grids, a common key is a tuple of coordinates, like `(row, col)`, as tuples are immutable. This is useful for storing visited locations or properties of a cell. (e.g., in Valid Sudoku).
- **Serializing Objects**: For more complex objects, like a tree node, you can create a string representation (serialize it) to use as a key. For example, to find duplicate subtrees, you could generate a string like `"(1, (2, N, N), (3, N, N))"` from a pre-order traversal to represent a subtree's structure and values. This string can then be used as a key in a hash map to count occurrences.

Designing the right key is a creative and crucial step in solving many hash table problems.
