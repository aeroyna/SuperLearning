# Graph Algorithms

## Learning Objectives
- Apply pathfinding algorithms
- Calculate centrality measures
- Detect communities in graphs
- Use similarity algorithms for recommendations

---

## 1. Graph Data Science Library

### Installation and Setup

```cypher
// Neo4j Graph Data Science (GDS) library provides graph algorithms

// Check if GDS is installed
RETURN gds.version()

// Create in-memory graph projection for algorithms
CALL gds.graph.project(
    'myGraph',                    // Graph name
    'Person',                     // Node labels
    'KNOWS',                      // Relationship types
    {
        nodeProperties: ['age'],
        relationshipProperties: ['strength']
    }
)

// List projected graphs
CALL gds.graph.list()

// Drop graph when done
CALL gds.graph.drop('myGraph')
```

### Algorithm Execution Modes

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Execution Modes                                   │
│                                                                      │
│  Mode        │ Purpose                                              │
│  ────────────┼──────────────────────────────────────────────────── │
│  stream      │ Returns results as stream (SELECT-like)              │
│  stats       │ Returns summary statistics only                      │
│  mutate      │ Writes results to in-memory graph                   │
│  write       │ Writes results back to Neo4j database               │
│                                                                      │
│  Example:                                                            │
│  gds.pageRank.stream()   -- Return PageRank scores                 │
│  gds.pageRank.stats()    -- Return stats (min, max, mean)          │
│  gds.pageRank.mutate()   -- Add score to projected graph           │
│  gds.pageRank.write()    -- Write score to database                │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Pathfinding Algorithms

### Shortest Path

```cypher
// Dijkstra's shortest path (weighted)
MATCH (source:City {name: 'London'})
MATCH (target:City {name: 'Paris'})
CALL gds.shortestPath.dijkstra.stream('cityGraph', {
    sourceNode: source,
    targetNode: target,
    relationshipWeightProperty: 'distance'
})
YIELD index, sourceNode, targetNode, totalCost, nodeIds, costs, path
RETURN
    gds.util.asNode(sourceNode).name AS from,
    gds.util.asNode(targetNode).name AS to,
    totalCost AS totalDistance,
    [nodeId IN nodeIds | gds.util.asNode(nodeId).name] AS route

// A* shortest path (with heuristic)
CALL gds.shortestPath.astar.stream('cityGraph', {
    sourceNode: source,
    targetNode: target,
    latitudeProperty: 'latitude',
    longitudeProperty: 'longitude',
    relationshipWeightProperty: 'distance'
})
YIELD totalCost, path
RETURN totalCost, path
```

### All Shortest Paths

```cypher
// Find all shortest paths between two nodes
MATCH (source:Person {name: 'Alice'})
MATCH (target:Person {name: 'Bob'})
CALL gds.allShortestPaths.dijkstra.stream('socialGraph', {
    sourceNode: source,
    targetNode: target
})
YIELD sourceNode, targetNode, distance, path
RETURN path, distance
```

### Single Source Shortest Path

```cypher
// Shortest paths from one node to all others
MATCH (source:Station {name: 'Central'})
CALL gds.allShortestPaths.delta.stream('transitGraph', {
    sourceNode: source,
    relationshipWeightProperty: 'travelTime'
})
YIELD targetNode, distance
RETURN gds.util.asNode(targetNode).name AS destination, distance
ORDER BY distance
```

### Breadth-First Search

```cypher
// BFS traversal
MATCH (source:Person {name: 'Alice'})
CALL gds.bfs.stream('socialGraph', {
    sourceNode: source,
    maxDepth: 3
})
YIELD path
RETURN path
```

---

## 3. Centrality Algorithms

### PageRank

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PageRank Algorithm                                │
│                                                                      │
│  Measures importance based on incoming links                        │
│  (Links from important pages count more)                            │
│                                                                      │
│        [A]────▶[B]◀────[C]                                          │
│         │       ▲       │                                            │
│         │       │       │                                            │
│         └──────▶[D]◀────┘                                            │
│                                                                      │
│  D has highest PageRank (most incoming links)                       │
│  B is second (link from high-ranked D)                              │
└─────────────────────────────────────────────────────────────────────┘
```

```cypher
// Calculate PageRank
CALL gds.pageRank.stream('webGraph', {
    maxIterations: 20,
    dampingFactor: 0.85
})
YIELD nodeId, score
RETURN gds.util.asNode(nodeId).name AS page, score
ORDER BY score DESC
LIMIT 10

// Write PageRank to nodes
CALL gds.pageRank.write('webGraph', {
    maxIterations: 20,
    writeProperty: 'pagerank'
})
YIELD nodePropertiesWritten, ranIterations
```

### Betweenness Centrality

```cypher
// Nodes that bridge different parts of the graph
CALL gds.betweenness.stream('socialGraph')
YIELD nodeId, score
RETURN gds.util.asNode(nodeId).name AS person, score
ORDER BY score DESC
LIMIT 10

// High betweenness = important connector/bridge
```

### Closeness Centrality

```cypher
// How close a node is to all other nodes
CALL gds.closeness.stream('socialGraph')
YIELD nodeId, score
RETURN gds.util.asNode(nodeId).name AS person, score
ORDER BY score DESC
LIMIT 10

// High closeness = can reach others quickly
```

### Degree Centrality

```cypher
// Simple count of connections
CALL gds.degree.stream('socialGraph', {
    orientation: 'UNDIRECTED'
})
YIELD nodeId, score
RETURN gds.util.asNode(nodeId).name AS person, score AS connections
ORDER BY connections DESC

// Orientation options: NATURAL, REVERSE, UNDIRECTED
```

### Article Rank

```cypher
// Variant of PageRank, lowers influence of low-degree nodes
CALL gds.articleRank.stream('citationGraph', {
    maxIterations: 20,
    dampingFactor: 0.85
})
YIELD nodeId, score
RETURN gds.util.asNode(nodeId).title AS paper, score
ORDER BY score DESC
```

---

## 4. Community Detection

### Louvain

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Community Detection                               │
│                                                                      │
│  Finds groups of densely connected nodes                            │
│                                                                      │
│    ┌─────────────────┐         ┌─────────────────┐                  │
│    │   Community A   │         │   Community B   │                  │
│    │  ┌───┐   ┌───┐  │         │  ┌───┐   ┌───┐  │                  │
│    │  │ 1 │───│ 2 │  │         │  │ 5 │───│ 6 │  │                  │
│    │  └───┘   └───┘  │  weak   │  └───┘   └───┘  │                  │
│    │     \   /       │◄───────▶│     \   /       │                  │
│    │      ┌───┐      │  link   │      ┌───┐      │                  │
│    │      │ 3 │──────│─────────│──────│ 7 │      │                  │
│    │      └───┘      │         │      └───┘      │                  │
│    │         │       │         │         │       │                  │
│    │      ┌───┐      │         │      ┌───┐      │                  │
│    │      │ 4 │      │         │      │ 8 │      │                  │
│    │      └───┘      │         │      └───┘      │                  │
│    └─────────────────┘         └─────────────────┘                  │
└─────────────────────────────────────────────────────────────────────┘
```

```cypher
// Louvain community detection
CALL gds.louvain.stream('socialGraph')
YIELD nodeId, communityId
RETURN communityId, collect(gds.util.asNode(nodeId).name) AS members
ORDER BY size(members) DESC

// With intermediate communities
CALL gds.louvain.stream('socialGraph', {
    includeIntermediateCommunities: true
})
YIELD nodeId, communityId, intermediateCommunityIds
RETURN nodeId, communityId, intermediateCommunityIds

// Write communities to nodes
CALL gds.louvain.write('socialGraph', {
    writeProperty: 'community'
})
YIELD communityCount, modularity
```

### Label Propagation

```cypher
// Fast community detection
CALL gds.labelPropagation.stream('socialGraph')
YIELD nodeId, communityId
RETURN communityId, collect(gds.util.asNode(nodeId).name) AS members
ORDER BY size(members) DESC
```

### Weakly Connected Components

```cypher
// Find disconnected subgraphs
CALL gds.wcc.stream('graph')
YIELD nodeId, componentId
RETURN componentId, count(*) AS size
ORDER BY size DESC

// Write component IDs
CALL gds.wcc.write('graph', {
    writeProperty: 'componentId'
})
```

### Strongly Connected Components

```cypher
// For directed graphs (cycles)
CALL gds.scc.stream('directedGraph')
YIELD nodeId, componentId
RETURN componentId, collect(gds.util.asNode(nodeId).name) AS members
```

### Triangle Count

```cypher
// Count triangles each node participates in
CALL gds.triangleCount.stream('socialGraph')
YIELD nodeId, triangleCount
RETURN gds.util.asNode(nodeId).name AS person, triangleCount
ORDER BY triangleCount DESC

// Local clustering coefficient
CALL gds.localClusteringCoefficient.stream('socialGraph')
YIELD nodeId, localClusteringCoefficient
RETURN gds.util.asNode(nodeId).name, localClusteringCoefficient
```

---

## 5. Similarity Algorithms

### Node Similarity (Jaccard)

```cypher
// Find similar nodes based on shared neighbors
CALL gds.nodeSimilarity.stream('purchaseGraph', {
    topK: 10,
    similarityCutoff: 0.5
})
YIELD node1, node2, similarity
RETURN
    gds.util.asNode(node1).name AS customer1,
    gds.util.asNode(node2).name AS customer2,
    similarity
ORDER BY similarity DESC

// Write similarity relationships
CALL gds.nodeSimilarity.write('purchaseGraph', {
    writeRelationshipType: 'SIMILAR_TO',
    writeProperty: 'score',
    topK: 5
})
```

### K-Nearest Neighbors (KNN)

```cypher
// Similarity based on node properties
CALL gds.knn.stream('productGraph', {
    topK: 5,
    nodeProperties: ['price', 'rating', 'category_embedding']
})
YIELD node1, node2, similarity
RETURN
    gds.util.asNode(node1).name AS product1,
    gds.util.asNode(node2).name AS product2,
    similarity
```

### Cosine Similarity

```cypher
// For vector embeddings
CALL gds.nodeSimilarity.stream('graph', {
    similarityMetric: 'COSINE'
})
YIELD node1, node2, similarity
RETURN node1, node2, similarity
```

---

## 6. Link Prediction

### Common Neighbors

```cypher
// Predict links based on shared neighbors
CALL gds.linkPrediction.commonNeighbors.stream({
    node1: source,
    node2: target,
    relationshipQuery: 'MATCH (n)-[:KNOWS]-(m) RETURN id(n) AS source, id(m) AS target'
})
YIELD node1, node2, score
RETURN score
```

### Preferential Attachment

```cypher
// High-degree nodes attract more links
CALL gds.linkPrediction.preferentialAttachment.stream({
    node1: source,
    node2: target
})
YIELD node1, node2, score
RETURN score
```

### Adamic Adar

```cypher
// Weighted common neighbors (rare neighbors count more)
CALL gds.linkPrediction.adamicAdar.stream({
    node1: source,
    node2: target
})
YIELD node1, node2, score
RETURN score
```

---

## 7. Path Analysis

### Random Walk

```cypher
// Generate random walks for embeddings
CALL gds.randomWalk.stream('graph', {
    walkLength: 80,
    walksPerNode: 10,
    sourceNodes: [1, 2, 3]
})
YIELD nodeIds
RETURN nodeIds
```

### Node2Vec

```cypher
// Generate node embeddings
CALL gds.node2vec.stream('graph', {
    embeddingDimension: 128,
    walkLength: 80,
    walksPerNode: 10,
    inOutFactor: 1.0,
    returnFactor: 1.0
})
YIELD nodeId, embedding
RETURN gds.util.asNode(nodeId).name, embedding
```

---

## 8. Practical Examples

### Fraud Detection

```cypher
// 1. Create graph projection
CALL gds.graph.project(
    'fraudGraph',
    ['Account', 'Person', 'Phone', 'Address'],
    ['OWNS', 'HAS_PHONE', 'LIVES_AT']
)

// 2. Find connected components (fraud rings)
CALL gds.wcc.stream('fraudGraph')
YIELD nodeId, componentId
WITH componentId, collect(nodeId) AS nodes
WHERE size(nodes) > 1
RETURN componentId, size(nodes) AS ringSize,
       [n IN nodes | gds.util.asNode(n).id] AS members
ORDER BY ringSize DESC

// 3. Calculate PageRank within fraud clusters
CALL gds.pageRank.stream('fraudGraph')
YIELD nodeId, score
WITH gds.util.asNode(nodeId) AS node, score
WHERE 'Account' IN labels(node)
RETURN node.id AS account, score AS riskScore
ORDER BY riskScore DESC
```

### Social Influence Analysis

```cypher
// Create social graph
CALL gds.graph.project(
    'socialInfluence',
    'User',
    'FOLLOWS'
)

// Find influencers with PageRank
CALL gds.pageRank.stream('socialInfluence')
YIELD nodeId, score
WITH gds.util.asNode(nodeId) AS user, score
RETURN user.username, score AS influence
ORDER BY influence DESC
LIMIT 20

// Find communities
CALL gds.louvain.write('socialInfluence', {
    writeProperty: 'community'
})

// Analyze community bridges
CALL gds.betweenness.stream('socialInfluence')
YIELD nodeId, score
WITH gds.util.asNode(nodeId) AS user, score
WHERE score > 1000
RETURN user.username, score AS bridgeScore
ORDER BY bridgeScore DESC
```

### Product Recommendations

```cypher
// Create purchase graph
CALL gds.graph.project(
    'recommendations',
    ['Customer', 'Product'],
    {
        PURCHASED: {
            type: 'PURCHASED',
            orientation: 'UNDIRECTED'
        }
    }
)

// Find similar customers
CALL gds.nodeSimilarity.stream('recommendations', {
    topK: 10
})
YIELD node1, node2, similarity
WITH gds.util.asNode(node1) AS customer1,
     gds.util.asNode(node2) AS customer2,
     similarity
WHERE 'Customer' IN labels(customer1)
RETURN customer1.id, customer2.id, similarity

// Recommend products
MATCH (me:Customer {id: 'C001'})-[:PURCHASED]->(myProducts)
MATCH (similar:Customer)-[:PURCHASED]->(rec:Product)
WHERE me <> similar
  AND NOT (me)-[:PURCHASED]->(rec)
WITH rec, count(*) AS score
RETURN rec.name, score
ORDER BY score DESC
LIMIT 10
```

---

## Summary

| Category | Algorithms |
|----------|------------|
| Pathfinding | Dijkstra, A*, BFS, DFS |
| Centrality | PageRank, Betweenness, Closeness, Degree |
| Community | Louvain, Label Propagation, WCC, SCC |
| Similarity | Jaccard, KNN, Cosine |
| Link Prediction | Common Neighbors, Adamic Adar |
| Embeddings | Node2Vec, Random Walk |

---

## Best Practices

```
Graph Projection:
✓ Project only needed data
✓ Use native projection for performance
✓ Drop projections when done
✓ Consider relationship orientation

Algorithm Selection:
✓ Use stream mode for exploration
✓ Use write mode for persisting results
✓ Tune parameters based on data
✓ Validate results with known examples

Performance:
✓ Profile algorithms before production
✓ Use sampling for large graphs
✓ Monitor memory usage
✓ Consider parallel execution
```
