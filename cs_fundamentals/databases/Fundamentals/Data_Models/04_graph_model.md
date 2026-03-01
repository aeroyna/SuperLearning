# Graph Model

## 1. Introduction

The **graph model** represents data as nodes (vertices) and relationships (edges), making it ideal for highly connected data where relationships are as important as the data itself.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         GRAPH MODEL STRUCTURE                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│                          ┌──────────────┐                                    │
│                          │    Alice     │                                    │
│                          │   (Person)   │                                    │
│                          │  age: 28     │                                    │
│                          └──────┬───────┘                                    │
│                                 │                                            │
│               WORKS_AT ─────────┼──────── KNOWS                              │
│              since: 2020        │        since: 2019                         │
│                                 │                                            │
│      ┌──────────────┐          │           ┌──────────────┐                 │
│      │   TechCorp   │          │           │     Bob      │                 │
│      │  (Company)   │◄─────────┘           │   (Person)   │                 │
│      │ employees:50 │                       │   age: 32    │                 │
│      └──────────────┘                       └──────────────┘                 │
│                                                                              │
│   NODE: Entity with labels and properties                                    │
│   RELATIONSHIP: Connection with type, direction, and properties             │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Core Concepts

### 2.1 Nodes (Vertices)

Nodes represent entities with labels and properties:

```cypher
// Neo4j Cypher - Create nodes
CREATE (alice:Person {name: 'Alice', age: 28, email: 'alice@example.com'})
CREATE (bob:Person {name: 'Bob', age: 32})
CREATE (techcorp:Company {name: 'TechCorp', industry: 'Technology', employees: 500})

// Node with multiple labels
CREATE (alice:Person:Developer:Manager {name: 'Alice'})

// Properties can be various types
CREATE (product:Product {
    name: 'Laptop',
    price: 999.99,
    tags: ['electronics', 'computer'],
    inStock: true,
    metadata: {category: 'tech'}
})
```

### 2.2 Relationships (Edges)

Relationships connect nodes with type, direction, and properties:

```cypher
// Create relationships
CREATE (alice)-[:KNOWS {since: 2019}]->(bob)
CREATE (alice)-[:WORKS_AT {since: 2020, role: 'Developer'}]->(techcorp)
CREATE (bob)-[:WORKS_AT {since: 2018, role: 'Manager'}]->(techcorp)

// Relationships are directional but can be traversed both ways
MATCH (a:Person)-[:KNOWS]-(b:Person)  // Either direction
MATCH (a:Person)-[:KNOWS]->(b:Person) // Specific direction
MATCH (a:Person)<-[:KNOWS]-(b:Person) // Reverse direction

// Multiple relationship types
CREATE (alice)-[:FRIEND_OF]->(bob)
CREATE (alice)-[:COLLEAGUE_OF]->(bob)
CREATE (alice)-[:MANAGES]->(bob)
```

### 2.3 Property Graph Model

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      PROPERTY GRAPH ELEMENTS                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   NODES                          RELATIONSHIPS                               │
│   ┌──────────────────┐           ─────────────────────                      │
│   │ • Unique ID      │           • Unique ID                                 │
│   │ • Labels (types) │           • Type (one per relationship)              │
│   │ • Properties     │           • Direction (start → end)                  │
│   └──────────────────┘           • Properties                               │
│                                                                              │
│   Labels: :Person, :Company      Types: :KNOWS, :WORKS_AT                   │
│   Properties: {name: "Alice"}    Properties: {since: 2020}                  │
│                                                                              │
│   INDEX-FREE ADJACENCY:                                                      │
│   Each node directly references its adjacent nodes                          │
│   No index lookup needed for traversal → O(1) per hop                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Cypher Query Language

### 3.1 Pattern Matching

```cypher
// Basic pattern matching
MATCH (p:Person) RETURN p

// Match with properties
MATCH (p:Person {name: 'Alice'}) RETURN p

// Match relationships
MATCH (p:Person)-[:WORKS_AT]->(c:Company)
RETURN p.name, c.name

// Variable length paths
MATCH (p:Person)-[:KNOWS*1..3]-(friend:Person)  // 1 to 3 hops
RETURN DISTINCT friend.name

// Any relationship type
MATCH (a)-[r]->(b) RETURN type(r)

// Optional match (like LEFT JOIN)
MATCH (p:Person)
OPTIONAL MATCH (p)-[:WORKS_AT]->(c:Company)
RETURN p.name, c.name
```

### 3.2 Filtering and Ordering

```cypher
// WHERE clause
MATCH (p:Person)
WHERE p.age >= 21 AND p.age <= 65
RETURN p.name, p.age

// Pattern in WHERE
MATCH (p:Person)
WHERE (p)-[:WORKS_AT]->(:Company {name: 'TechCorp'})
RETURN p.name

// NOT pattern
MATCH (p:Person)
WHERE NOT (p)-[:KNOWS]->(:Person {name: 'Alice'})
RETURN p.name

// IN operator
MATCH (p:Person)
WHERE p.name IN ['Alice', 'Bob', 'Charlie']
RETURN p

// String matching
MATCH (p:Person)
WHERE p.name STARTS WITH 'A'
   OR p.email CONTAINS '@techcorp'
   OR p.name =~ '.*son$'  // Regex
RETURN p

// ORDER BY and LIMIT
MATCH (p:Person)
RETURN p.name, p.age
ORDER BY p.age DESC
LIMIT 10
```

### 3.3 Aggregation

```cypher
// Count
MATCH (p:Person)-[:WORKS_AT]->(c:Company)
RETURN c.name, count(p) as employee_count
ORDER BY employee_count DESC

// Collect (aggregate to list)
MATCH (p:Person)-[:KNOWS]->(friend:Person)
RETURN p.name, collect(friend.name) as friends

// Multiple aggregations
MATCH (p:Person)
RETURN
    count(p) as total,
    avg(p.age) as avg_age,
    min(p.age) as min_age,
    max(p.age) as max_age

// Group by relationship
MATCH (p:Person)-[r:WORKS_AT]->(c:Company)
RETURN c.name,
       count(p) as employees,
       collect(p.name) as employee_names,
       avg(date().year - r.since) as avg_tenure
```

### 3.4 Create, Update, Delete

```cypher
// Create nodes and relationships
CREATE (alice:Person {name: 'Alice', age: 28})
CREATE (bob:Person {name: 'Bob', age: 32})
CREATE (alice)-[:KNOWS {since: 2019}]->(bob)

// MERGE (create if not exists)
MERGE (p:Person {email: 'alice@example.com'})
ON CREATE SET p.name = 'Alice', p.createdAt = datetime()
ON MATCH SET p.lastSeen = datetime()
RETURN p

// Update properties
MATCH (p:Person {name: 'Alice'})
SET p.age = 29, p.updatedAt = datetime()
RETURN p

// Remove properties
MATCH (p:Person {name: 'Alice'})
REMOVE p.temporaryField
RETURN p

// Delete nodes and relationships
MATCH (p:Person {name: 'Alice'})-[r]-()
DELETE r  // Must delete relationships first
DELETE p

// Delete all (DETACH deletes relationships too)
MATCH (p:Person {name: 'Alice'})
DETACH DELETE p
```

### 3.5 Path Operations

```cypher
// Shortest path
MATCH path = shortestPath(
    (alice:Person {name: 'Alice'})-[:KNOWS*]-(bob:Person {name: 'Bob'})
)
RETURN path, length(path)

// All shortest paths
MATCH paths = allShortestPaths(
    (alice:Person {name: 'Alice'})-[:KNOWS*]-(bob:Person {name: 'Bob'})
)
RETURN paths

// Path with conditions
MATCH path = (start:Person)-[:KNOWS*1..5]->(end:Person)
WHERE start.name = 'Alice'
  AND ALL(n IN nodes(path) WHERE n.age >= 21)
RETURN path

// Extract path elements
MATCH path = (a:Person)-[:KNOWS*]->(b:Person)
RETURN
    nodes(path) as people,
    relationships(path) as connections,
    length(path) as hops
```

---

## 4. Common Use Cases

### 4.1 Social Network

```cypher
// Friend recommendations (friends of friends)
MATCH (user:Person {name: 'Alice'})-[:FRIEND_OF]->(:Person)-[:FRIEND_OF]->(recommended:Person)
WHERE NOT (user)-[:FRIEND_OF]->(recommended)
  AND user <> recommended
RETURN recommended.name, count(*) as mutual_friends
ORDER BY mutual_friends DESC
LIMIT 5

// Mutual friends between two users
MATCH (user1:Person {name: 'Alice'})-[:FRIEND_OF]->(mutual:Person)<-[:FRIEND_OF]-(user2:Person {name: 'Bob'})
RETURN mutual.name

// Influence score (followers count)
MATCH (p:Person)
OPTIONAL MATCH (p)<-[:FOLLOWS]-(follower)
RETURN p.name, count(follower) as followers
ORDER BY followers DESC

// Find communities (connected components)
MATCH (p:Person)-[:FRIEND_OF*]-(connected:Person)
WITH p, collect(DISTINCT connected) as community
RETURN p.name, size(community) as community_size
```

### 4.2 Fraud Detection

```cypher
// Find accounts sharing multiple identifiers
MATCH (a1:Account)-[:HAS_PHONE]->(phone:Phone)<-[:HAS_PHONE]-(a2:Account)
WHERE a1 <> a2
WITH a1, a2, count(phone) as shared_phones
WHERE shared_phones > 1
MATCH (a1)-[:HAS_EMAIL]->(email:Email)<-[:HAS_EMAIL]-(a2)
WITH a1, a2, shared_phones, count(email) as shared_emails
WHERE shared_phones + shared_emails >= 3
RETURN a1.id, a2.id, shared_phones, shared_emails

// Transaction chain analysis
MATCH path = (source:Account {flagged: true})-[:TRANSFERRED_TO*1..5]->(destination:Account)
WHERE destination.balance > 10000
RETURN path, reduce(total = 0, r IN relationships(path) | total + r.amount) as total_transferred

// Ring detection (money laundering patterns)
MATCH path = (a:Account)-[:TRANSFERRED_TO*3..7]->(a)
WHERE ALL(r IN relationships(path) WHERE r.amount > 1000)
RETURN path
```

### 4.3 Knowledge Graph

```cypher
// Create knowledge graph
CREATE (python:Language {name: 'Python'})
CREATE (django:Framework {name: 'Django'})
CREATE (flask:Framework {name: 'Flask'})
CREATE (web:Domain {name: 'Web Development'})

CREATE (django)-[:WRITTEN_IN]->(python)
CREATE (flask)-[:WRITTEN_IN]->(python)
CREATE (django)-[:USED_FOR]->(web)
CREATE (flask)-[:USED_FOR]->(web)

// Query: Find all technologies for a domain
MATCH (tech)-[:USED_FOR]->(:Domain {name: 'Web Development'})
OPTIONAL MATCH (tech)-[:WRITTEN_IN]->(lang:Language)
RETURN tech.name, collect(lang.name) as languages

// Semantic search: Technologies related to Python
MATCH (python:Language {name: 'Python'})<-[:WRITTEN_IN]-(tech)
MATCH (tech)-[:USED_FOR]->(domain)
RETURN tech.name, collect(DISTINCT domain.name) as use_cases
```

### 4.4 Recommendation Engine

```cypher
// Product recommendations based on purchase history
MATCH (user:Customer {id: 'cust123'})-[:PURCHASED]->(product:Product)
      <-[:PURCHASED]-(other:Customer)-[:PURCHASED]->(recommended:Product)
WHERE NOT (user)-[:PURCHASED]->(recommended)
RETURN recommended.name, count(DISTINCT other) as purchase_score
ORDER BY purchase_score DESC
LIMIT 10

// Content-based recommendations
MATCH (user:Customer)-[:PURCHASED]->(p:Product)-[:IN_CATEGORY]->(cat:Category)
WITH user, cat, count(p) as purchase_count
ORDER BY purchase_count DESC
WITH user, collect(cat)[0..3] as top_categories
MATCH (recommended:Product)-[:IN_CATEGORY]->(cat)
WHERE cat IN top_categories AND NOT (user)-[:PURCHASED]->(recommended)
RETURN DISTINCT recommended.name
LIMIT 10

// Collaborative filtering with ratings
MATCH (user:Customer {id: 'cust123'})-[r1:RATED]->(product:Product)
      <-[r2:RATED]-(similar:Customer)
WHERE abs(r1.score - r2.score) <= 1
WITH user, similar, count(product) as common_products
WHERE common_products >= 3
MATCH (similar)-[r:RATED]->(recommended:Product)
WHERE NOT (user)-[:RATED]->(recommended) AND r.score >= 4
RETURN recommended.name, avg(r.score) as predicted_score
ORDER BY predicted_score DESC
```

### 4.5 Access Control / Authorization

```cypher
// Check if user has permission
MATCH (user:User {id: 'user123'})-[:MEMBER_OF*1..]->(group:Group)
      -[:HAS_PERMISSION]->(perm:Permission {name: 'edit_document'})
RETURN count(perm) > 0 as has_permission

// Get all user permissions (including inherited)
MATCH (user:User {id: 'user123'})-[:MEMBER_OF*0..]->(group)
      -[:HAS_PERMISSION]->(perm:Permission)
RETURN DISTINCT perm.name as permission

// Role hierarchy
MATCH path = (user:User)-[:MEMBER_OF*]->(topGroup:Group)
WHERE NOT (topGroup)-[:MEMBER_OF]->()
RETURN user.name, [g IN nodes(path) WHERE g:Group | g.name] as group_hierarchy
```

---

## 5. Graph Algorithms

### 5.1 Centrality Algorithms

```cypher
// Degree centrality (number of connections)
MATCH (p:Person)
RETURN p.name,
       size((p)-[:KNOWS]->()) as outgoing,
       size((p)<-[:KNOWS]-()) as incoming,
       size((p)-[:KNOWS]-()) as total

// PageRank (influence/importance)
CALL gds.pageRank.stream('myGraph')
YIELD nodeId, score
RETURN gds.util.asNode(nodeId).name AS name, score
ORDER BY score DESC

// Betweenness centrality (bridge nodes)
CALL gds.betweenness.stream('myGraph')
YIELD nodeId, score
RETURN gds.util.asNode(nodeId).name AS name, score
ORDER BY score DESC
```

### 5.2 Community Detection

```cypher
// Louvain community detection
CALL gds.louvain.stream('myGraph')
YIELD nodeId, communityId
RETURN communityId, collect(gds.util.asNode(nodeId).name) as members
ORDER BY size(members) DESC

// Label propagation
CALL gds.labelPropagation.stream('myGraph')
YIELD nodeId, communityId
RETURN communityId, count(*) as size
ORDER BY size DESC
```

### 5.3 Path Finding

```cypher
// Dijkstra's shortest path (weighted)
CALL gds.shortestPath.dijkstra.stream('myGraph', {
    sourceNode: startNode,
    targetNode: endNode,
    relationshipWeightProperty: 'distance'
})
YIELD path, totalCost
RETURN path, totalCost

// A* algorithm
CALL gds.shortestPath.astar.stream('myGraph', {
    sourceNode: startNode,
    targetNode: endNode,
    latitudeProperty: 'lat',
    longitudeProperty: 'lon',
    relationshipWeightProperty: 'distance'
})
YIELD path, totalCost
RETURN path, totalCost
```

---

## 6. Code Examples Across Languages

### Python (neo4j driver)

```python
from neo4j import GraphDatabase

class Neo4jConnection:
    def __init__(self, uri, user, password):
        self.driver = GraphDatabase.driver(uri, auth=(user, password))

    def close(self):
        self.driver.close()

    def create_person(self, name, age):
        with self.driver.session() as session:
            result = session.execute_write(
                self._create_person, name, age
            )
            return result

    @staticmethod
    def _create_person(tx, name, age):
        query = """
        CREATE (p:Person {name: $name, age: $age})
        RETURN p
        """
        result = tx.run(query, name=name, age=age)
        return result.single()[0]

    def find_friends(self, name):
        with self.driver.session() as session:
            result = session.execute_read(
                self._find_friends, name
            )
            return result

    @staticmethod
    def _find_friends(tx, name):
        query = """
        MATCH (p:Person {name: $name})-[:KNOWS]->(friend:Person)
        RETURN friend.name as name, friend.age as age
        """
        result = tx.run(query, name=name)
        return [{"name": record["name"], "age": record["age"]}
                for record in result]

    def friend_recommendations(self, name, limit=5):
        with self.driver.session() as session:
            query = """
            MATCH (user:Person {name: $name})-[:KNOWS]->(:Person)-[:KNOWS]->(recommended:Person)
            WHERE NOT (user)-[:KNOWS]->(recommended) AND user <> recommended
            RETURN recommended.name as name, count(*) as mutual_friends
            ORDER BY mutual_friends DESC
            LIMIT $limit
            """
            result = session.run(query, name=name, limit=limit)
            return [dict(record) for record in result]

# Usage
conn = Neo4jConnection("bolt://localhost:7687", "neo4j", "password")
conn.create_person("Alice", 28)
friends = conn.find_friends("Alice")
recommendations = conn.friend_recommendations("Alice")
conn.close()
```

### Java (Neo4j Driver)

```java
import org.neo4j.driver.*;
import java.util.*;

public class Neo4jExample implements AutoCloseable {
    private final Driver driver;

    public Neo4jExample(String uri, String user, String password) {
        driver = GraphDatabase.driver(uri, AuthTokens.basic(user, password));
    }

    @Override
    public void close() {
        driver.close();
    }

    public void createPerson(String name, int age) {
        try (Session session = driver.session()) {
            session.executeWrite(tx -> {
                tx.run("CREATE (p:Person {name: $name, age: $age})",
                       Map.of("name", name, "age", age));
                return null;
            });
        }
    }

    public List<Map<String, Object>> findFriends(String name) {
        try (Session session = driver.session()) {
            return session.executeRead(tx -> {
                Result result = tx.run("""
                    MATCH (p:Person {name: $name})-[:KNOWS]->(friend:Person)
                    RETURN friend.name as name, friend.age as age
                    """, Map.of("name", name));

                List<Map<String, Object>> friends = new ArrayList<>();
                while (result.hasNext()) {
                    Record record = result.next();
                    friends.add(Map.of(
                        "name", record.get("name").asString(),
                        "age", record.get("age").asInt()
                    ));
                }
                return friends;
            });
        }
    }

    public static void main(String[] args) {
        try (Neo4jExample app = new Neo4jExample("bolt://localhost:7687", "neo4j", "password")) {
            app.createPerson("Alice", 28);
            List<Map<String, Object>> friends = app.findFriends("Alice");
            friends.forEach(System.out::println);
        }
    }
}
```

### JavaScript (neo4j-driver)

```javascript
const neo4j = require('neo4j-driver');

class Neo4jConnection {
    constructor(uri, user, password) {
        this.driver = neo4j.driver(uri, neo4j.auth.basic(user, password));
    }

    async close() {
        await this.driver.close();
    }

    async createPerson(name, age) {
        const session = this.driver.session();
        try {
            const result = await session.executeWrite(async tx => {
                const query = `
                    CREATE (p:Person {name: $name, age: $age})
                    RETURN p
                `;
                return await tx.run(query, { name, age });
            });
            return result.records[0].get('p').properties;
        } finally {
            await session.close();
        }
    }

    async findFriends(name) {
        const session = this.driver.session();
        try {
            const result = await session.executeRead(async tx => {
                const query = `
                    MATCH (p:Person {name: $name})-[:KNOWS]->(friend:Person)
                    RETURN friend.name as name, friend.age as age
                `;
                return await tx.run(query, { name });
            });
            return result.records.map(record => ({
                name: record.get('name'),
                age: record.get('age').toNumber()
            }));
        } finally {
            await session.close();
        }
    }

    async friendRecommendations(name, limit = 5) {
        const session = this.driver.session();
        try {
            const result = await session.executeRead(async tx => {
                const query = `
                    MATCH (user:Person {name: $name})-[:KNOWS]->(:Person)-[:KNOWS]->(recommended:Person)
                    WHERE NOT (user)-[:KNOWS]->(recommended) AND user <> recommended
                    RETURN recommended.name as name, count(*) as mutualFriends
                    ORDER BY mutualFriends DESC
                    LIMIT $limit
                `;
                return await tx.run(query, { name, limit: neo4j.int(limit) });
            });
            return result.records.map(record => ({
                name: record.get('name'),
                mutualFriends: record.get('mutualFriends').toNumber()
            }));
        } finally {
            await session.close();
        }
    }
}

// Usage
async function main() {
    const conn = new Neo4jConnection('bolt://localhost:7687', 'neo4j', 'password');
    await conn.createPerson('Alice', 28);
    const friends = await conn.findFriends('Alice');
    console.log(friends);
    await conn.close();
}

main().catch(console.error);
```

---

## 7. Advantages and Limitations

### Advantages

| Advantage | Description |
|-----------|-------------|
| **Relationship Performance** | O(1) traversal via index-free adjacency |
| **Intuitive Model** | Natural representation of connected data |
| **Flexible Schema** | Easy to add new node types and relationships |
| **Pattern Matching** | Powerful query language for complex patterns |
| **Graph Algorithms** | Built-in support for centrality, community detection |

### Limitations

| Limitation | Description |
|------------|-------------|
| **Aggregate Queries** | Not optimized for counting/summing across all data |
| **Sharding Complexity** | Graph partitioning is challenging |
| **Learning Curve** | Requires different thinking than relational |
| **Full Scans** | Queries without start nodes can be slow |
| **Write Scaling** | Heavy writes can bottleneck |

---

## 8. Summary

Graph databases excel when:
- Relationships between entities are primary concern
- Queries involve traversing connections (friends, paths, recommendations)
- Data is naturally networked (social, knowledge, fraud, access control)
- Schema evolves with new relationship types

Use graph databases for:
- Social networks and recommendations
- Fraud detection and security
- Knowledge graphs and semantic search
- Network and IT operations
- Master data management
