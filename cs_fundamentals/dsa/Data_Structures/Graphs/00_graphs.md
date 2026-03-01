## Introduction to Graphs

A graph is a non-linear data structure used to represent relationships between objects. It consists of a set of **nodes** (or vertices) and a set of **edges** that connect pairs of nodes. Graphs are one of the most flexible and widely applicable data structures, modeling everything from social networks to road systems to the world wide web.

### Core Idea & Analogy
Unlike trees, graphs are a more general structure. They do not have a designated `root` node, and they do not have a strict parent-child hierarchy. A node can be connected to any other node, and these connections can be either one-way or two-way.

- **Analogy 1: Social Network**. Think of Facebook. Every person is a node, and a "friendship" is an edge connecting two people. This is an **undirected** graph because friendship is mutual.
- **Analogy 2: Street Map**. Think of a city map. Intersections are nodes, and the streets connecting them are edges. If a street is one-way, it's a **directed** edge; otherwise, it's an **undirected** edge.

### Key Terminology
- **Node (Vertex)**: A single entity in the graph.
- **Edge**: A connection between two nodes.
- **Directed vs. Undirected Graph**:
  - **Undirected**: Edges are two-way. If A is connected to B, B is also connected to A.
  - **Directed (Digraph)**: Edges are one-way, represented by arrows. An edge from A to B does not imply an edge from B to A.
- **Neighbors**: Two nodes are neighbors if they are connected by an edge.
- **Degree (of a node)**:
  - In an undirected graph, the number of edges connected to the node.
  - In a directed graph:
    - **Indegree**: Number of incoming edges.
    - **Outdegree**: Number of outgoing edges.
- **Path**: A sequence of nodes connected by edges.
- **Cycle**: A path that starts and ends at the same node. A graph with no cycles is **acyclic**. A **Directed Acyclic Graph (DAG)** is a very common structure with important applications like task scheduling.
- **Connected Components**: In an undirected graph, a subset of nodes where every node is reachable from every other node in the subset. A graph can have one or multiple connected components.

### Practice
- [Practice Problems](Practice_Problems/00_practice_problems.md)