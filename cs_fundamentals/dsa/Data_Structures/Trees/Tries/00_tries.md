## Tries (Prefix Trees)

A **Trie**, also known as a **prefix tree** or digital tree, is a tree-like data structure specialized for storing and retrieving strings. It's particularly efficient for tasks that involve string prefixes.

### Core Idea
Unlike other trees where a node might store an entire key, nodes in a trie do not store keys. Instead, a node's position within the tree represents the key with which it is associated. Each path from the root to a node represents a prefix. The children of a node represent the next possible characters in the strings.

For example, to store "car" and "cat":
- The `root` would have a child for `c`.
- The `c` node would have a child for `a`.
- The `a` node would have two children, one for `r` and one for `t`.

To distinguish between a prefix that is also a complete word (like "car") and a prefix that is not (like "ca"), nodes often contain a boolean flag, `isEndOfWord`.

### Analogy
A trie works like an autocomplete system. As you type a prefix, like "comp", the system navigates a trie of words down the path `c` -> `o` -> `m` -> `p`. From that node, it can immediately suggest all possible completions, such as "computer", "complete", and "compare", because they all share that common prefix path in the trie.

### Key Characteristics
- **Fast Prefix Operations**: Searching for a word or checking if any word starts with a given prefix takes O(L) time, where L is the length of the word or prefix. This is independent of the number of words stored in the trie.
- **Space Efficiency for Shared Prefixes**: Words with common prefixes (like "tree", "trie", "trip") share the initial part of their structure in the trie, which can save space.
- **Alphabet Dependency**: The space and time complexity can depend on the size of the alphabet. Each node may have to store pointers for every possible character.


### Practice
- [Practice Problems](Practice_Problems/00_practice_problems.md)