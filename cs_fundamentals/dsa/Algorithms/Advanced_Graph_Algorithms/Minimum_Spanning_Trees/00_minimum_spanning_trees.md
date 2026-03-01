## Introduction to Minimum Spanning Trees (MST)

Given a **connected, undirected, and weighted graph**, a Minimum Spanning Tree (MST) is a subgraph that connects all the vertices together with the minimum possible total edge weight, without forming any cycles.

### Core Idea
Imagine you are tasked with connecting a set of towns with a fiber optic cable network. The cost to lay cable between any two towns varies. Your goal is to connect all the towns (so that there is a path from any town to any other) using the minimum possible amount of cable. The resulting network would be a Minimum Spanning Tree.

A "spanning tree" of a graph must satisfy three properties:
1.  It is a **subgraph** that includes all the vertices of the original graph.
2.  It is **connected**.
3.  It is **acyclic** (it's a tree).

A **Minimum Spanning Tree** is a spanning tree with the additional property that the sum of the weights of its edges is less than or equal to the sum of the weights of every other spanning tree.

### Key Properties
- For a graph with `V` vertices, any spanning tree (including an MST) will have exactly `V-1` edges.
- An MST is not necessarily unique. A graph can have multiple different MSTs if it has multiple edges with the same weight.
- If all edge weights in a graph are unique, then the graph will have only one, unique MST.

### Classic Algorithms
There are two main greedy algorithms used to find the MST of a graph. Both are considered standard and important to know.

1.  **Kruskal's Algorithm**: This algorithm builds the MST by iteratively adding the "safest" edge. It sorts all edges by weight and adds the next-cheapest edge as long as it does not form a cycle. It uses the Union-Find data structure to efficiently detect cycles.

2.  **Prim's Algorithm**: This algorithm builds the MST by growing a single tree. It starts from an arbitrary vertex and, at each step, adds the cheapest possible edge that connects a vertex in the growing tree to a vertex outside the tree. It is very similar to Dijkstra's algorithm and is often implemented with a priority queue.

Both algorithms are "greedy" because they make a locally optimal choice at each step (i.e., picking the cheapest available edge), and both are guaranteed to find a globally optimal solution (the MST).
