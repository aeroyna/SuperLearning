# Graphs

A graph is a non-linear data structure consisting of a set of **vertices** (or nodes) and a set of **edges** that connect pairs of vertices.

## Terminology

*   **Vertex:** A node in the graph.
*   **Edge:** A link between two vertices.
*   **Directed Graph:** A graph where the edges have a direction.
*   **Undirected Graph:** A graph where the edges do not have a direction.
*   **Weighted Graph:** A graph where each edge has a weight or cost associated with it.
*   **Path:** A sequence of vertices connected by edges.
*   **Cycle:** A path that starts and ends at the same vertex.

## Graph Representation

There are two common ways to represent a graph:

### 1. Adjacency Matrix

An adjacency matrix is a 2D array where `adj[i][j] = 1` if there is an edge from vertex `i` to vertex `j`, and `0` otherwise. For a weighted graph, `adj[i][j]` can store the weight of the edge.

*   **Pros:** Fast to check if an edge exists between two vertices (O(1)).
*   **Cons:** Uses a lot of memory (O(V<sup>2</sup>)), which is inefficient for sparse graphs (graphs with few edges).

### 2. Adjacency List

An adjacency list is an array of lists. `adj[i]` contains a list of all the vertices that are adjacent to vertex `i`.

*   **Pros:** More memory efficient for sparse graphs.
*   **Cons:** Slower to check if an edge exists between two vertices (O(log k) or O(k), where k is the number of neighbors).

In C++, an adjacency list can be represented as a `std::vector<std::vector<int>>` or `std::vector<std::list<int>>`.

### Example: Adjacency List

```cpp
#include <iostream>
#include <vector>
#include <list>

class Graph {
private:
    int num_vertices;
    std::vector<std::list<int>> adj;

public:
    Graph(int V) : num_vertices(V), adj(V) {}

    void add_edge(int u, int v) {
        adj[u].push_back(v);
        // For an undirected graph, add the reverse edge as well
        // adj[v].push_back(u);
    }

    void print() {
        for (int i = 0; i < num_vertices; ++i) {
            std::cout << "Adjacency list of vertex " << i << ": ";
            for (int neighbor : adj[i]) {
                std::cout << neighbor << " ";
            }
            std::cout << std::endl;
        }
    }
};

int main() {
    Graph g(4);
    g.add_edge(0, 1);
    g.add_edge(0, 2);
    g.add_edge(1, 2);
    g.add_edge(2, 0);
    g.add_edge(2, 3);
    g.add_edge(3, 3);
    g.print();
    return 0;
}
```

## Graph Traversal Algorithms

### Breadth-First Search (BFS)

BFS is an algorithm for traversing or searching a graph. It starts at a given vertex and explores all the neighbor nodes at the present depth prior to moving on to the nodes at the next depth level. BFS uses a queue.

### Depth-First Search (DFS)

DFS is an algorithm for traversing or searching a graph. It starts at a given vertex and explores as far as possible along each branch before backtracking. DFS uses a stack (or recursion).

## Common Graph Algorithms

*   **Dijkstra's Algorithm:** Finds the shortest path between two vertices in a weighted graph with non-negative edge weights.
*   **Bellman-Ford Algorithm:** Finds the shortest path in a weighted graph, even with negative edge weights.
*   **Floyd-Warshall Algorithm:** Finds the shortest paths between all pairs of vertices in a weighted graph.
*   **Kruskal's Algorithm / Prim's Algorithm:** Finds the Minimum Spanning Tree (MST) of a weighted, undirected graph.
*   **Topological Sort:** A linear ordering of the vertices of a directed acyclic graph (DAG).
