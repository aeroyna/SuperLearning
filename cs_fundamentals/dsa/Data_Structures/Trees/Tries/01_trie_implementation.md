## Trie Implementation

A Trie can be implemented using two main classes:
1.  `TrieNode`: Represents a single node in the trie. It contains a way to access its children and a flag to mark the end of a word.
2.  `Trie`: The main class that holds the `root` of the trie and orchestrates the main operations like `insert`, `search`, and `startsWith`.

### 1. The TrieNode Class
Each node needs to store references to its children. A hash map (or dictionary in Python) is a perfect choice for this, where keys are the characters and values are the child `TrieNode` objects.

>[!example]- C++
>```cpp
>class TrieNode {
>public:
>    // A map to hold children nodes.
>    // e.g., {'a': TrieNode*, 'b': TrieNode*}
>    unordered_map<char, TrieNode*> children;
>
>    // A boolean flag to indicate if a word ends at this node.
>    bool isEndOfWord;
>
>    TrieNode() {
>        isEndOfWord = false;
>    }
>};
>```

>[!example]- Java
>```java
>class TrieNode {
>    // A HashMap to hold children nodes.
>    // e.g., {'a': TrieNode, 'b': TrieNode}
>    Map<Character, TrieNode> children;
>
>    // A boolean flag to indicate if a word ends at this node.
>    boolean isEndOfWord;
>
>    public TrieNode() {
>        children = new HashMap<>();
>        isEndOfWord = false;
>    }
>}
>```

>[!example]- Python
>```python
>class TrieNode:
>    """A node in the trie structure."""
>    def __init__(self):
>        # A dictionary to hold children nodes.
>        # e.g., {'a': TrieNode(), 'b': TrieNode()}
>        self.children = {}
>
>        # A boolean flag to indicate if a word ends at this node.
>        self.isEndOfWord = False
>```

>[!example]- JavaScript
>```javascript
>class TrieNode {
>    constructor() {
>        // A Map to hold children nodes.
>        // e.g., {'a': TrieNode, 'b': TrieNode}
>        this.children = new Map();
>
>        // A boolean flag to indicate if a word ends at this node.
>        this.isEndOfWord = false;
>    }
>}
>```

### 2. The Trie Class
This class ties everything together. It initializes the trie with an empty root node and provides the methods for interacting with the data structure.

>[!example]- C++
>```cpp
>class Trie {
>private:
>    TrieNode* root;
>
>public:
>    /**
>     * Initializes the trie with an empty root node.
>     */
>    Trie() {
>        root = new TrieNode();
>    }
>
>    /**
>     * Inserts a word into the trie.
>     * Time Complexity: O(L), where L is the length of the word.
>     * Space Complexity: O(L) in the worst case (for a new word with no shared prefix).
>     */
>    void insert(string word) {
>        TrieNode* curr = root;
>        for (char c : word) {
>            // If the character path doesn't exist, create a new node.
>            if (curr->children.find(c) == curr->children.end()) {
>                curr->children[c] = new TrieNode();
>            }
>            // Move down the trie.
>            curr = curr->children[c];
>        }
>        // After the loop, mark the current node as the end of a word.
>        curr->isEndOfWord = true;
>    }
>
>    /**
>     * Returns true if the exact word exists in the trie.
>     * Time Complexity: O(L).
>     */
>    bool search(string word) {
>        TrieNode* curr = root;
>        for (char c : word) {
>            if (curr->children.find(c) == curr->children.end()) {
>                return false;
>            }
>            curr = curr->children[c];
>        }
>        // The word only exists if the full path is found AND it's marked as a word.
>        return curr->isEndOfWord;
>    }
>
>    /**
>     * Returns true if there is any word in the trie that starts with the given prefix.
>     * Time Complexity: O(L), where L is the length of the prefix.
>     */
>    bool startsWith(string prefix) {
>        TrieNode* curr = root;
>        for (char c : prefix) {
>            if (curr->children.find(c) == curr->children.end()) {
>                return false;
>            }
>            curr = curr->children[c];
>        }
>        // If the full prefix path exists, return true.
>        return true;
>    }
>};
>```

>[!example]- Java
>```java
>class Trie {
>    private TrieNode root;
>
>    /**
>     * Initializes the trie with an empty root node.
>     */
>    public Trie() {
>        root = new TrieNode();
>    }
>
>    /**
>     * Inserts a word into the trie.
>     * Time Complexity: O(L), where L is the length of the word.
>     * Space Complexity: O(L) in the worst case (for a new word with no shared prefix).
>     */
>    public void insert(String word) {
>        TrieNode curr = root;
>        for (char c : word.toCharArray()) {
>            // If the character path doesn't exist, create a new node.
>            if (!curr.children.containsKey(c)) {
>                curr.children.put(c, new TrieNode());
>            }
>            // Move down the trie.
>            curr = curr.children.get(c);
>        }
>        // After the loop, mark the current node as the end of a word.
>        curr.isEndOfWord = true;
>    }
>
>    /**
>     * Returns true if the exact word exists in the trie.
>     * Time Complexity: O(L).
>     */
>    public boolean search(String word) {
>        TrieNode curr = root;
>        for (char c : word.toCharArray()) {
>            if (!curr.children.containsKey(c)) {
>                return false;
>            }
>            curr = curr.children.get(c);
>        }
>        // The word only exists if the full path is found AND it's marked as a word.
>        return curr.isEndOfWord;
>    }
>
>    /**
>     * Returns true if there is any word in the trie that starts with the given prefix.
>     * Time Complexity: O(L), where L is the length of the prefix.
>     */
>    public boolean startsWith(String prefix) {
>        TrieNode curr = root;
>        for (char c : prefix.toCharArray()) {
>            if (!curr.children.containsKey(c)) {
>                return false;
>            }
>            curr = curr.children.get(c);
>        }
>        // If the full prefix path exists, return true.
>        return true;
>    }
>}
>```

>[!example]- Python
>```python
>class Trie:
>    """The Trie data structure."""
>    def __init__(self):
>        """Initializes the trie with an empty root node."""
>        self.root = TrieNode()
>
>    def insert(self, word: str) -> None:
>        """
>        Inserts a word into the trie.
>        Time Complexity: O(L), where L is the length of the word.
>        Space Complexity: O(L) in the worst case (for a new word with no shared prefix).
>        """
>        curr = self.root
>        for char in word:
>            # If the character path doesn't exist, create a new node.
>            if char not in curr.children:
>                curr.children[char] = TrieNode()
>            # Move down the trie.
>            curr = curr.children[char]
>        # After the loop, mark the current node as the end of a word.
>        curr.isEndOfWord = True
>
>    def search(self, word: str) -> bool:
>        """
>        Returns True if the exact word exists in the trie.
>        Time Complexity: O(L).
>        """
>        curr = self.root
>        for char in word:
>            if char not in curr.children:
>                return False
>            curr = curr.children[char]
>        # The word only exists if the full path is found AND it's marked as a word.
>        return curr.isEndOfWord
>
>    def startsWith(self, prefix: str) -> bool:
>        """
>        Returns True if there is any word in the trie that starts with the given prefix.
>        Time Complexity: O(L), where L is the length of the prefix.
>        """
>        curr = self.root
>        for char in prefix:
>            if char not in curr.children:
>                return False
>            curr = curr.children[char]
>        # If the full prefix path exists, return True.
>        return True
>
>```

>[!example]- JavaScript
>```javascript
>class Trie {
>    /**
>     * Initializes the trie with an empty root node.
>     */
>    constructor() {
>        this.root = new TrieNode();
>    }
>
>    /**
>     * Inserts a word into the trie.
>     * Time Complexity: O(L), where L is the length of the word.
>     * Space Complexity: O(L) in the worst case (for a new word with no shared prefix).
>     */
>    insert(word) {
>        let curr = this.root;
>        for (const char of word) {
>            // If the character path doesn't exist, create a new node.
>            if (!curr.children.has(char)) {
>                curr.children.set(char, new TrieNode());
>            }
>            // Move down the trie.
>            curr = curr.children.get(char);
>        }
>        // After the loop, mark the current node as the end of a word.
>        curr.isEndOfWord = true;
>    }
>
>    /**
>     * Returns true if the exact word exists in the trie.
>     * Time Complexity: O(L).
>     */
>    search(word) {
>        let curr = this.root;
>        for (const char of word) {
>            if (!curr.children.has(char)) {
>                return false;
>            }
>            curr = curr.children.get(char);
>        }
>        // The word only exists if the full path is found AND it's marked as a word.
>        return curr.isEndOfWord;
>    }
>
>    /**
>     * Returns true if there is any word in the trie that starts with the given prefix.
>     * Time Complexity: O(L), where L is the length of the prefix.
>     */
>    startsWith(prefix) {
>        let curr = this.root;
>        for (const char of prefix) {
>            if (!curr.children.has(char)) {
>                return false;
>            }
>            curr = curr.children.get(char);
>        }
>        // If the full prefix path exists, return true.
>        return true;
>    }
>}
>```

This implementation is the standard structure expected in coding interviews for Trie-related problems (e.g., LeetCode's "Implement Trie (Prefix Tree)").
