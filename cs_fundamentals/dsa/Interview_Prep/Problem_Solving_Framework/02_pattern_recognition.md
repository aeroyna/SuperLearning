## Pattern Recognition in Problem Solving

The single most effective skill for passing coding interviews is **pattern recognition**. Interview problems are rarely completely unique; most are variations of a few dozen fundamental patterns. Instead of trying to memorize solutions to hundreds of individual problems, the goal is to learn the underlying patterns. When you see a new problem, you can then map it to a known pattern and adapt its solution template.

### Why Patterns Matter
- **Speed**: Recognizing a pattern quickly allows you to bypass the initial "how do I even start?" phase and immediately begin formulating a high-level approach.
- **Confidence**: Knowing that a problem fits a known pattern gives you a clear path forward, reducing interview anxiety.
- **Optimization**: Patterns often come with well-understood optimal solutions. If you identify a problem as a "Top K" problem, you immediately know that a heap is likely the best tool, rather than starting with a brute-force sorting approach.

### How to Develop Pattern Recognition
1.  **Focus on Categories**: When you solve a problem, don't just understand the solution. Categorize it. Is this a sliding window problem? A graph traversal? A binary search on the answer?
2.  **Ask "Why?"**: For a given problem, why is a particular data structure or algorithm the right choice? What properties of the problem make it suitable for a heap, a hash map, or a greedy approach? Understanding the "why" is more important than memorizing the "how".
3.  **Active Recall**: After learning a pattern, actively try to recall 2-3 different problems that fit that pattern. For example, after learning about the Two Heaps pattern for finding a median, recognize that it can also be used for problems involving splitting a set into two balanced halves.

### A High-Level "Mental Flowchart" for a New Problem

1.  **Analyze the Input & Output:**
    - Is the input a sorted array? → *Think Binary Search, Two Pointers.*
    - Is the input a collection of strings that need to be grouped? → *Think Hash Map with a custom key (e.g., sorted string for anagrams).*
    - Is the input a grid/matrix? → *Think of it as an Implicit Graph. Consider BFS/DFS.*
    - Does the problem involve prerequisites or dependencies? → *Think Topological Sort.*

2.  **Analyze the Keywords & Constraints:**
    - "Shortest path," "fewest moves," "minimum steps" in an unweighted graph/grid? → **BFS**.
    - "Shortest path" in a weighted graph with no negative edges? → **Dijkstra's**.
    - "Longest/shortest subarray/substring" that meets a condition? → **Sliding Window**.
    - "Top/Largest/Smallest K" elements? → **Heap (Priority Queue)**.
    - "Find all permutations/combinations/subsets"? → **Backtracking**.
    - "Minimize the maximum" or "Maximize the minimum"? → **Binary Search on the Answer**.
    - "Count the number of ways" to do something? → **Dynamic Programming**.
    - A very small input size (e.g., `n <= 20`)? → *Hints at an exponential complexity solution like Backtracking or DP with Bitmasking.*
    - Need to check for duplicates or count frequencies? → **Hash Set / Hash Map**.
    - Involves merging or checking for overlapping intervals? → **Sort the intervals** (by start or end), then iterate.

By consciously thinking in terms of these patterns, you build a mental framework that makes new, unseen problems feel familiar and solvable.
