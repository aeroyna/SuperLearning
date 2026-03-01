## Introduction to Topological Sort

Topological Sort is a linear ordering of the vertices in a **Directed Acyclic Graph (DAG)**. For every directed edge from vertex `u` to vertex `v`, vertex `u` must come before vertex `v` in the ordering.

### The Core Idea
Think of a set of tasks where some tasks have prerequisites. For example, to put on your shoes, you must first put on your socks. A topological sort gives you a valid sequence in which to perform these tasks. If you can't put on your socks because you need to put on your shoes first, you have a cycle, and a valid ordering is impossible.

This leads to the most important prerequisite for topological sorting: it is **only possible on a Directed Acyclic Graph (DAG)**. If a graph contains a cycle (e.g., A -> B -> C -> A), no valid topological ordering exists because there is no "first" vertex. Therefore, the ability to perform a topological sort is also a test for whether a directed graph is acyclic.

### Key Properties
- **Application**: The canonical use case is **task scheduling** or **dependency resolution**. This includes course prerequisites in a curriculum, build dependencies in a compiler, or resolving symbol dependencies in a linker.
- **Non-uniqueness**: A graph can have multiple valid topological sorts. For example, if task A is a prerequisite for B and C, both `A, B, C` and `A, C, B` could be valid orderings.
- **Start and End Points**: A DAG will always have at least one vertex with an **indegree** of 0 (no incoming edges) and at least one vertex with an **outdegree** of 0 (no outgoing edges). These serve as the natural starting and ending points for the sort.

### Algorithms
There are two standard algorithms for performing a topological sort:

1.  **Kahn's Algorithm**: This is a BFS-based approach. It works by finding nodes with no incoming edges (indegree of 0), adding them to the sorted list, and then "removing" them and their outgoing edges from the graph. It repeats this process until no nodes are left.

2.  **DFS-Based Algorithm**: This approach uses Depth-First Search. As the DFS traversal for a node and all its descendants finishes, that node is added to the *front* of the sorted list. The node that finishes last is the first node in the topological order.

Both algorithms efficiently produce a valid topological ordering and can also be used to detect cycles in a directed graph.
