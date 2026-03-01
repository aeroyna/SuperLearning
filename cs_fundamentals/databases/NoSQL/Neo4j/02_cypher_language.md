# Cypher Language

## Learning Objectives
- Master Cypher syntax for graph queries
- Perform CRUD operations on nodes and relationships
- Write complex pattern matching queries
- Use aggregations, filtering, and path operations

---

## 1. Cypher Fundamentals

### ASCII Art Patterns

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Cypher Pattern Syntax                             │
│                                                                      │
│  Nodes:                                                              │
│  ()              Anonymous node                                      │
│  (n)             Node with variable n                                │
│  (:Person)       Node with label                                     │
│  (p:Person)      Node with variable and label                       │
│  (p:Person {name: 'Alice'})   With properties                       │
│                                                                      │
│  Relationships:                                                      │
│  -->             Outgoing relationship (anonymous)                   │
│  -[r]->          With variable                                       │
│  -[:KNOWS]->     With type                                           │
│  -[r:KNOWS]->    With variable and type                             │
│  -[r:KNOWS {since: 2020}]->   With properties                       │
│  <--             Incoming relationship                               │
│  --              Undirected (either direction)                       │
│                                                                      │
│  Paths:                                                              │
│  (a)-[r]->(b)               Single hop                              │
│  (a)-[*2]->(b)              Exactly 2 hops                          │
│  (a)-[*1..3]->(b)           1 to 3 hops                             │
│  (a)-[*]->(b)               Any number of hops                      │
│  path = (a)-[*]->(b)        Store path in variable                  │
└─────────────────────────────────────────────────────────────────────┘
```

### Basic Query Structure

```cypher
// MATCH: Find patterns in the graph
MATCH (p:Person)
RETURN p

// WHERE: Filter results
MATCH (p:Person)
WHERE p.age > 21
RETURN p.name

// ORDER BY: Sort results
MATCH (p:Person)
RETURN p.name, p.age
ORDER BY p.age DESC

// LIMIT and SKIP: Pagination
MATCH (p:Person)
RETURN p.name
ORDER BY p.name
SKIP 10
LIMIT 10

// WITH: Chain query parts (piping)
MATCH (p:Person)-[:POSTED]->(post)
WITH p, count(post) AS postCount
WHERE postCount > 5
RETURN p.name, postCount
```

---

## 2. CRUD Operations

### CREATE

```cypher
// Create node
CREATE (p:Person {name: 'Alice', age: 30})
RETURN p

// Create multiple nodes
CREATE (a:Person {name: 'Alice'}),
       (b:Person {name: 'Bob'}),
       (c:Person {name: 'Carol'})

// Create relationship
MATCH (a:Person {name: 'Alice'})
MATCH (b:Person {name: 'Bob'})
CREATE (a)-[:KNOWS {since: 2020}]->(b)

// Create node and relationship together
CREATE (a:Person {name: 'Alice'})-[:KNOWS]->(b:Person {name: 'Bob'})

// Create with RETURN
CREATE (p:Person {name: 'New Person'})
RETURN p, id(p) AS nodeId
```

### READ (MATCH)

```cypher
// Match all nodes
MATCH (n) RETURN n LIMIT 100

// Match by label
MATCH (p:Person) RETURN p

// Match by property
MATCH (p:Person {name: 'Alice'}) RETURN p

// Match with WHERE
MATCH (p:Person)
WHERE p.age >= 18 AND p.age <= 65
RETURN p

// Match relationship
MATCH (a:Person)-[r:KNOWS]->(b:Person)
RETURN a.name, type(r), b.name

// Match path
MATCH path = (a:Person)-[:KNOWS*1..3]->(b:Person)
RETURN path
```

### UPDATE

```cypher
// Set property
MATCH (p:Person {name: 'Alice'})
SET p.age = 31
RETURN p

// Set multiple properties
MATCH (p:Person {name: 'Alice'})
SET p.age = 31, p.email = 'alice@example.com'
RETURN p

// Replace all properties
MATCH (p:Person {name: 'Alice'})
SET p = {name: 'Alice Smith', age: 31}
// Removes all existing properties!

// Add properties (keep existing)
MATCH (p:Person {name: 'Alice'})
SET p += {nickname: 'Ali', verified: true}

// Add label
MATCH (p:Person {name: 'Alice'})
SET p:Employee

// Remove property
MATCH (p:Person {name: 'Alice'})
REMOVE p.nickname

// Remove label
MATCH (p:Person {name: 'Alice'})
REMOVE p:Employee
```

### DELETE

```cypher
// Delete node (must have no relationships)
MATCH (p:Person {name: 'Temp'})
DELETE p

// Delete node and all relationships
MATCH (p:Person {name: 'Temp'})
DETACH DELETE p

// Delete relationship only
MATCH (a:Person)-[r:KNOWS]->(b:Person)
WHERE a.name = 'Alice' AND b.name = 'Bob'
DELETE r

// Delete all (use with caution!)
MATCH (n) DETACH DELETE n
```

### MERGE (Upsert)

```cypher
// Create if not exists
MERGE (p:Person {email: 'alice@example.com'})
RETURN p

// With ON CREATE
MERGE (p:Person {email: 'alice@example.com'})
ON CREATE SET p.name = 'Alice', p.createdAt = datetime()
RETURN p

// With ON MATCH
MERGE (p:Person {email: 'alice@example.com'})
ON MATCH SET p.lastSeen = datetime()
RETURN p

// Both ON CREATE and ON MATCH
MERGE (p:Person {email: 'alice@example.com'})
ON CREATE SET p.createdAt = datetime()
ON MATCH SET p.lastSeen = datetime()
RETURN p

// MERGE relationship
MATCH (a:Person {name: 'Alice'})
MATCH (b:Person {name: 'Bob'})
MERGE (a)-[r:KNOWS]->(b)
ON CREATE SET r.since = date()
RETURN r
```

---

## 3. Pattern Matching

### Complex Patterns

```cypher
// Friends of friends
MATCH (me:Person {name: 'Alice'})-[:KNOWS]->(friend)-[:KNOWS]->(fof)
WHERE fof <> me AND NOT (me)-[:KNOWS]->(fof)
RETURN DISTINCT fof.name

// Triangle pattern
MATCH (a:Person)-[:KNOWS]->(b:Person)-[:KNOWS]->(c:Person)-[:KNOWS]->(a)
RETURN a, b, c

// Optional match (left outer join)
MATCH (p:Person)
OPTIONAL MATCH (p)-[:HAS_PHONE]->(phone:Phone)
RETURN p.name, phone.number

// Multiple patterns
MATCH (a:Person)-[:WORKS_AT]->(company:Company)
MATCH (a)-[:LIVES_IN]->(city:City)
RETURN a.name, company.name, city.name
```

### Variable Length Paths

```cypher
// Exactly N hops
MATCH (a:Person)-[:KNOWS*3]->(b:Person)
RETURN a, b

// Range of hops
MATCH (a:Person)-[:KNOWS*1..5]->(b:Person)
RETURN a, b

// Zero or more (includes self)
MATCH (a:Person)-[:KNOWS*0..]->(b:Person)
RETURN a, b

// Any length (dangerous without limit!)
MATCH (a:Person)-[:KNOWS*]->(b:Person)
RETURN a, b
LIMIT 100

// Named path
MATCH path = (a:Person)-[:KNOWS*]->(b:Person)
RETURN path, length(path)

// Path with conditions
MATCH path = (a:Person)-[:KNOWS*1..5]->(b:Person)
WHERE all(r IN relationships(path) WHERE r.strength > 0.5)
RETURN path
```

### Shortest Path

```cypher
// Single shortest path
MATCH path = shortestPath((a:Person {name: 'Alice'})-[*]-(b:Person {name: 'Charlie'}))
RETURN path, length(path)

// All shortest paths
MATCH paths = allShortestPaths((a:Person {name: 'Alice'})-[*]-(b:Person {name: 'Charlie'}))
RETURN paths

// With relationship type filter
MATCH path = shortestPath((a:Person {name: 'Alice'})-[:KNOWS|WORKS_WITH*]-(b:Person {name: 'Charlie'}))
RETURN path

// With path conditions
MATCH path = shortestPath((a:Person)-[*]-(b:Person))
WHERE a.name = 'Alice' AND b.name = 'Charlie'
  AND all(node IN nodes(path) WHERE node.active = true)
RETURN path
```

---

## 4. Filtering and Conditions

### WHERE Clause

```cypher
// Comparison operators
MATCH (p:Person)
WHERE p.age > 21
  AND p.age <= 65
  AND p.name <> 'Bob'
RETURN p

// String operations
MATCH (p:Person)
WHERE p.name STARTS WITH 'Al'
   OR p.name ENDS WITH 'ice'
   OR p.name CONTAINS 'li'
RETURN p

// Regular expressions
MATCH (p:Person)
WHERE p.email =~ '.*@gmail\\.com'
RETURN p

// List membership
MATCH (p:Person)
WHERE p.country IN ['USA', 'Canada', 'Mexico']
RETURN p

// NULL checks
MATCH (p:Person)
WHERE p.email IS NOT NULL
RETURN p

// Pattern existence
MATCH (p:Person)
WHERE EXISTS { (p)-[:PURCHASED]->(:Product) }
RETURN p

// NOT pattern
MATCH (p:Person)
WHERE NOT (p)-[:PURCHASED]->(:Product {category: 'Electronics'})
RETURN p
```

### List Predicates

```cypher
// ALL - every element must satisfy
MATCH path = (a)-[*]->(b)
WHERE all(node IN nodes(path) WHERE node.active = true)
RETURN path

// ANY - at least one must satisfy
MATCH (p:Person)
WHERE any(skill IN p.skills WHERE skill STARTS WITH 'Java')
RETURN p

// NONE - no element satisfies
MATCH path = (a)-[*]->(b)
WHERE none(r IN relationships(path) WHERE r.blocked = true)
RETURN path

// SINGLE - exactly one satisfies
MATCH (p:Person)-[r:MANAGES]->(team:Team)
WHERE single(m IN team.members WHERE m.role = 'lead')
RETURN p
```

---

## 5. Aggregation

### Aggregate Functions

```cypher
// Count
MATCH (p:Person)
RETURN count(p) AS personCount

// Count distinct
MATCH (p:Person)-[:VISITED]->(city:City)
RETURN p.name, count(DISTINCT city) AS citiesVisited

// Sum, Avg, Min, Max
MATCH (o:Order)-[:CONTAINS]->(item)
RETURN o.orderId,
       sum(item.price) AS total,
       avg(item.price) AS avgPrice,
       min(item.price) AS cheapest,
       max(item.price) AS mostExpensive

// Collect (aggregate to list)
MATCH (p:Person)-[:KNOWS]->(friend)
RETURN p.name, collect(friend.name) AS friends

// Collect with limit
MATCH (p:Person)-[:KNOWS]->(friend)
RETURN p.name, collect(friend.name)[0..5] AS topFriends
```

### Grouping with WITH

```cypher
// Group by
MATCH (p:Person)-[:PURCHASED]->(product)
WITH p, count(product) AS purchaseCount, sum(product.price) AS totalSpent
WHERE purchaseCount > 5
RETURN p.name, purchaseCount, totalSpent
ORDER BY totalSpent DESC

// Multiple aggregations
MATCH (c:Customer)-[:PLACED]->(o:Order)
WITH c.region AS region,
     count(o) AS orderCount,
     sum(o.total) AS revenue
RETURN region, orderCount, revenue
ORDER BY revenue DESC
```

---

## 6. List and Map Operations

### List Functions

```cypher
// Create list
RETURN [1, 2, 3, 4, 5] AS numbers
RETURN range(1, 10) AS oneToTen
RETURN range(0, 10, 2) AS evens

// List access
WITH [1, 2, 3, 4, 5] AS list
RETURN list[0] AS first,    // 1
       list[-1] AS last,    // 5
       list[1..3] AS slice  // [2, 3]

// List operations
RETURN size([1, 2, 3]) AS length        // 3
RETURN head([1, 2, 3])                  // 1
RETURN tail([1, 2, 3])                  // [2, 3]
RETURN last([1, 2, 3])                  // 3
RETURN reverse([1, 2, 3])               // [3, 2, 1]

// List comprehension
MATCH (p:Person)
RETURN [friend IN [(p)-[:KNOWS]->(f) | f.name] WHERE friend STARTS WITH 'A'] AS aFriends

// Reduce
WITH [1, 2, 3, 4, 5] AS numbers
RETURN reduce(total = 0, n IN numbers | total + n) AS sum
```

### Map Operations

```cypher
// Create map
RETURN {name: 'Alice', age: 30} AS person

// Map access
WITH {name: 'Alice', age: 30} AS person
RETURN person.name, person['age']

// Map from node
MATCH (p:Person {name: 'Alice'})
RETURN properties(p) AS allProperties

// Keys and values
RETURN keys({a: 1, b: 2}) AS keyList    // ['a', 'b']

// Map projection
MATCH (p:Person)
RETURN p {.name, .age, status: 'active'}
```

---

## 7. UNWIND and FOREACH

### UNWIND

```cypher
// Expand list to rows
UNWIND [1, 2, 3] AS num
RETURN num

// Create multiple nodes from list
UNWIND ['Alice', 'Bob', 'Carol'] AS name
CREATE (p:Person {name: name})
RETURN p

// Join with UNWIND
MATCH (p:Person)
UNWIND p.skills AS skill
RETURN skill, count(*) AS peopleWithSkill
ORDER BY peopleWithSkill DESC

// Nested UNWIND
UNWIND [[1, 2], [3, 4]] AS list
UNWIND list AS item
RETURN item
```

### FOREACH

```cypher
// Update in loop (side effects only, no RETURN)
MATCH path = (a:Person)-[:KNOWS*]->(b:Person)
FOREACH (n IN nodes(path) | SET n.visited = true)

// Create sequence
FOREACH (i IN range(1, 5) |
    CREATE (:Item {number: i})
)

// Conditional in FOREACH (using CASE)
MATCH (p:Person)
FOREACH (x IN CASE WHEN p.age > 18 THEN [1] ELSE [] END |
    SET p:Adult
)
```

---

## 8. Subqueries

### CALL Subquery

```cypher
// Correlated subquery
MATCH (p:Person)
CALL {
    WITH p
    MATCH (p)-[:PURCHASED]->(product)
    RETURN count(product) AS purchaseCount
}
RETURN p.name, purchaseCount

// EXISTS subquery
MATCH (p:Person)
WHERE EXISTS {
    MATCH (p)-[:PURCHASED]->(:Product {category: 'Electronics'})
}
RETURN p

// COUNT subquery
MATCH (p:Person)
WHERE COUNT {
    (p)-[:PURCHASED]->()
} > 5
RETURN p

// UNION in subquery
MATCH (p:Person)
CALL {
    WITH p
    MATCH (p)-[:FRIEND]->(f)
    RETURN f AS connection
    UNION
    MATCH (p)-[:COLLEAGUE]->(c)
    RETURN c AS connection
}
RETURN p.name, collect(connection.name) AS connections
```

---

## 9. Practical Examples

### Recommendation Query

```cypher
// "People who bought this also bought"
MATCH (p:Person)-[:PURCHASED]->(product:Product {name: 'Widget'})
MATCH (p)-[:PURCHASED]->(otherProduct)
WHERE otherProduct <> product
RETURN otherProduct.name, count(*) AS score
ORDER BY score DESC
LIMIT 5
```

### Fraud Detection Pattern

```cypher
// Find circular money transfers
MATCH path = (a:Account)-[:TRANSFERRED*3..6]->(a)
WHERE all(t IN relationships(path) WHERE t.amount > 10000)
RETURN path, [t IN relationships(path) | t.amount] AS amounts
```

### Social Network Influence

```cypher
// Find influencers (most followers)
MATCH (influencer:User)<-[:FOLLOWS]-(follower)
WITH influencer, count(follower) AS followers
WHERE followers > 1000
RETURN influencer.username, followers
ORDER BY followers DESC
LIMIT 10
```

---

## Summary

| Clause | Purpose |
|--------|---------|
| MATCH | Find patterns |
| CREATE | Create nodes/relationships |
| MERGE | Create if not exists |
| SET | Update properties |
| DELETE | Remove nodes/relationships |
| RETURN | Output results |
| WITH | Chain queries |
| WHERE | Filter results |
| ORDER BY | Sort results |
| UNWIND | Expand lists |

---

## Best Practices

```
Query Writing:
✓ Use parameters ($param) instead of literals
✓ Create indexes for frequently matched properties
✓ Use LIMIT with variable-length paths
✓ Prefer specific patterns over generic

Performance:
✓ Match on indexed properties first
✓ Filter early with WHERE
✓ Use PROFILE to analyze queries
✓ Avoid cartesian products (use WITH)

Readability:
✓ Use consistent naming conventions
✓ Break complex queries with WITH
✓ Comment complex patterns
✓ Use meaningful variable names
```
