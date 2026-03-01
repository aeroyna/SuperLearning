# Graph Concepts

## Learning Objectives
- Understand property graph model fundamentals
- Design effective graph schemas
- Apply graph modeling patterns
- Avoid common modeling anti-patterns

---

## 1. Property Graph Model

### Core Components

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Property Graph Elements                           │
│                                                                      │
│  NODES (Entities)                                                    │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                                                              │    │
│  │    ┌───────────────────────┐                                │    │
│  │    │ :Person:Employee      │ ← Multiple labels              │    │
│  │    │                       │                                │    │
│  │    │ name: "Alice Smith"   │ ← Properties                   │    │
│  │    │ age: 30               │   (key: value pairs)           │    │
│  │    │ email: "alice@co.com" │                                │    │
│  │    │ skills: ["Java",      │ ← Arrays allowed               │    │
│  │    │          "Python"]    │                                │    │
│  │    │                       │                                │    │
│  │    └───────────────────────┘                                │    │
│  │                                                              │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  RELATIONSHIPS (Connections)                                         │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                                                              │    │
│  │  ┌───────┐         ─[:WORKS_AT]─▶        ┌───────────┐     │    │
│  │  │ Alice │                               │  Company  │     │    │
│  │  └───────┘         since: 2020           └───────────┘     │    │
│  │                    role: "Engineer"                         │    │
│  │                    ↑                                        │    │
│  │     Relationships have:                                     │    │
│  │     • Type (one only, e.g., WORKS_AT)                      │    │
│  │     • Direction (start → end)                               │    │
│  │     • Properties (optional)                                 │    │
│  │                                                              │    │
│  └─────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
```

### Labels and Types

```cypher
// Labels categorize nodes (like tables in SQL)
CREATE (p:Person:Customer {name: 'Alice'})
// Node can have multiple labels

// Relationship types define connection meaning
CREATE (p)-[:PURCHASED {date: date()}]->(product)
CREATE (p)-[:REVIEWED {rating: 5}]->(product)
CREATE (p)-[:WISHLISTED]->(product)
// Relationships have exactly one type

// Query by label
MATCH (p:Person) RETURN p

// Query by relationship type
MATCH ()-[r:PURCHASED]->() RETURN r
```

### Properties

```cypher
// Supported property types:
// - String: "hello"
// - Integer: 42
// - Float: 3.14
// - Boolean: true, false
// - Date: date('2024-01-15')
// - DateTime: datetime()
// - Duration: duration('P1D')
// - Point: point({latitude: 40.7, longitude: -73.9})
// - Arrays: ["a", "b", "c"], [1, 2, 3]

// Set properties on nodes
CREATE (p:Person {
    name: 'Alice',
    born: date('1990-05-15'),
    active: true,
    score: 85.5,
    tags: ['developer', 'speaker'],
    location: point({latitude: 40.7128, longitude: -74.0060})
})

// Set properties on relationships
CREATE (a)-[:KNOWS {
    since: date('2020-01-01'),
    strength: 0.8,
    context: 'work'
}]->(b)
```

---

## 2. Graph Modeling Principles

### Whiteboard-Friendly

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Modeling Process                                  │
│                                                                      │
│  1. Identify Entities → Nodes                                       │
│     "What are the things in my domain?"                             │
│     Person, Company, Product, Order                                 │
│                                                                      │
│  2. Identify Connections → Relationships                            │
│     "How are these things connected?"                               │
│     Person WORKS_AT Company                                         │
│     Person PURCHASED Product                                        │
│     Order CONTAINS Product                                          │
│                                                                      │
│  3. Identify Attributes → Properties                                │
│     "What do I need to know about each?"                            │
│     Person: name, email, age                                        │
│     PURCHASED: date, quantity, price                                │
│                                                                      │
│  4. Identify Categories → Labels                                    │
│     "What types/categories exist?"                                  │
│     Person: Customer, Employee, Vendor                              │
│                                                                      │
│  Key Insight:                                                        │
│  If you can draw it on a whiteboard, you can model it in Neo4j     │
└─────────────────────────────────────────────────────────────────────┘
```

### Entity vs Relationship

```
┌─────────────────────────────────────────────────────────────────────┐
│                    When to Use Node vs Relationship                  │
│                                                                      │
│  Use NODE when:                                                      │
│  • Entity can exist independently                                   │
│  • Entity can connect to multiple other entities                    │
│  • You need to find/query the entity directly                       │
│  • The "thing" has complex attributes                               │
│                                                                      │
│  Use RELATIONSHIP when:                                              │
│  • Connection only exists between specific nodes                    │
│  • It represents an action/verb                                     │
│  • Simple attributes (or none)                                      │
│                                                                      │
│  Example: Email                                                      │
│                                                                      │
│  Option A: Relationship                                              │
│  (Alice)-[:EMAILED {subject: "Hi", date: ...}]->(Bob)               │
│  Good for: Simple email tracking                                    │
│                                                                      │
│  Option B: Node (Intermediate)                                       │
│  (Alice)-[:SENT]->(Email)-[:TO]->(Bob)                              │
│            ↓                                                         │
│     (cc)-[:CC]->(Email)                                             │
│     (Email)-[:HAS_ATTACHMENT]->(File)                               │
│  Good for: Complex email with CC, attachments, threading            │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3. Common Modeling Patterns

### Social Network

```cypher
// User follows User
CREATE (alice:User {username: 'alice'})
CREATE (bob:User {username: 'bob'})
CREATE (alice)-[:FOLLOWS {since: date()}]->(bob)

// Bidirectional friendship
CREATE (alice)-[:FRIENDS_WITH]->(bob)
CREATE (bob)-[:FRIENDS_WITH]->(alice)
// Or use single relationship, ignore direction in query

// User posts Content
CREATE (post:Post {content: '...', createdAt: datetime()})
CREATE (alice)-[:POSTED]->(post)

// User likes Content
CREATE (alice)-[:LIKES {at: datetime()}]->(post)

// User comments on Content
CREATE (comment:Comment {text: '...', at: datetime()})
CREATE (alice)-[:COMMENTED]->(comment)-[:ON]->(post)
```

### E-Commerce

```cypher
// Product catalog
CREATE (product:Product {
    sku: 'ABC123',
    name: 'Widget',
    price: 29.99
})

CREATE (category:Category {name: 'Electronics'})
CREATE (product)-[:IN_CATEGORY]->(category)

// Customer orders
CREATE (customer:Customer {email: 'john@example.com'})
CREATE (order:Order {orderId: 'ORD-001', date: date()})
CREATE (customer)-[:PLACED]->(order)
CREATE (order)-[:CONTAINS {quantity: 2, price: 29.99}]->(product)

// Reviews
CREATE (customer)-[:REVIEWED {rating: 5, text: 'Great!'}]->(product)

// Recommendations
MATCH (c:Customer)-[:PURCHASED]->(p:Product)<-[:PURCHASED]-(other:Customer)
MATCH (other)-[:PURCHASED]->(rec:Product)
WHERE NOT (c)-[:PURCHASED]->(rec)
RETURN rec, count(*) AS score
ORDER BY score DESC
```

### Knowledge Graph

```cypher
// Entities
CREATE (python:Language {name: 'Python'})
CREATE (guido:Person {name: 'Guido van Rossum'})
CREATE (google:Company {name: 'Google'})
CREATE (ai:Topic {name: 'Artificial Intelligence'})

// Relationships
CREATE (guido)-[:CREATED]->(python)
CREATE (guido)-[:WORKED_AT]->(google)
CREATE (python)-[:USED_IN]->(ai)
CREATE (google)-[:INVESTS_IN]->(ai)

// Query: What's connected to Python?
MATCH (python:Language {name: 'Python'})-[r]-(connected)
RETURN type(r), labels(connected), connected.name
```

### Hierarchical Data

```cypher
// Organization hierarchy
CREATE (ceo:Employee {name: 'CEO'})
CREATE (vp1:Employee {name: 'VP Engineering'})
CREATE (vp2:Employee {name: 'VP Sales'})
CREATE (eng1:Employee {name: 'Engineer 1'})

CREATE (ceo)-[:MANAGES]->(vp1)
CREATE (ceo)-[:MANAGES]->(vp2)
CREATE (vp1)-[:MANAGES]->(eng1)

// Find all reports (variable length path)
MATCH (ceo:Employee {name: 'CEO'})-[:MANAGES*]->(report)
RETURN report.name

// Find management chain
MATCH path = (emp:Employee {name: 'Engineer 1'})<-[:MANAGES*]-(manager)
RETURN [node IN nodes(path) | node.name] AS chain
```

### Time-Based Events

```cypher
// Event chain
CREATE (e1:Event {type: 'login', at: datetime()})
CREATE (e2:Event {type: 'view', at: datetime()})
CREATE (e3:Event {type: 'purchase', at: datetime()})

CREATE (user:User {id: 'u1'})
CREATE (user)-[:DID]->(e1)
CREATE (user)-[:DID]->(e2)
CREATE (user)-[:DID]->(e3)

// Or linked list pattern for ordering
CREATE (e1)-[:NEXT]->(e2)-[:NEXT]->(e3)

// Query user journey
MATCH (u:User {id: 'u1'})-[:DID]->(event)
RETURN event.type, event.at
ORDER BY event.at
```

---

## 4. Graph Model vs Relational Model

### Comparison

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Modeling Comparison                               │
│                                                                      │
│  Relational Model:                                                   │
│  ┌──────────────────┐  ┌──────────────────┐  ┌─────────────────┐   │
│  │ users            │  │ user_groups      │  │ groups          │   │
│  │ ────────────     │  │ ────────────     │  │ ────────────    │   │
│  │ id     INT       │  │ user_id  INT     │  │ id     INT      │   │
│  │ name   VARCHAR   │  │ group_id INT     │  │ name   VARCHAR  │   │
│  │ email  VARCHAR   │  │ role     VARCHAR │  │ type   VARCHAR  │   │
│  └──────────────────┘  └──────────────────┘  └─────────────────┘   │
│                                                                      │
│  Graph Model:                                                        │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                                                              │   │
│  │     ┌───────────────┐                                       │   │
│  │     │ :User         │                                       │   │
│  │     │ name: "Alice" │         ┌───────────────────┐         │   │
│  │     │ email: "..."  │─────────▶│ :Group            │         │   │
│  │     └───────────────┘ MEMBER  │ name: "Admins"    │         │   │
│  │                       role:   │ type: "security"  │         │   │
│  │                       "admin" └───────────────────┘         │   │
│  │                                                              │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
│  Key Differences:                                                    │
│  • No join table needed (relationship IS the connection)           │
│  • Relationship properties replace join table columns              │
│  • Schema is flexible (add properties anytime)                     │
│  • Navigation is direct (no table lookups)                         │
└─────────────────────────────────────────────────────────────────────┘
```

### Migration Thinking

```cypher
// Relational foreign keys become relationships
// SQL: orders.customer_id → customers.id
// Graph: (Customer)-[:PLACED]->(Order)

// Many-to-many join tables become direct relationships
// SQL: user_roles (user_id, role_id)
// Graph: (User)-[:HAS_ROLE]->(Role)

// Self-referencing tables become relationships
// SQL: employees (id, manager_id)
// Graph: (Employee)-[:REPORTS_TO]->(Manager:Employee)

// Hierarchies are natural
// SQL: categories (id, parent_id) with recursive CTEs
// Graph: (Category)-[:PARENT]->(ParentCategory)
//        (Child)-[:CHILD_OF*]->(Ancestor)
```

---

## 5. Anti-Patterns

### Rich Relationship vs Intermediate Node

```cypher
// Anti-pattern: Too many properties on relationship
CREATE (a)-[:TRANSACTION {
    amount: 100,
    currency: 'USD',
    date: date(),
    status: 'completed',
    merchant_name: '...',
    merchant_category: '...',
    fraud_score: 0.1,
    authorization_code: '...',
    // ... 20 more properties
}]->(b)

// Better: Intermediate node when relationship is complex
CREATE (a)-[:SENT]->(t:Transaction {
    id: 'TX001',
    amount: 100,
    // ... all properties
})-[:TO]->(b)
CREATE (t)-[:AT]->(merchant:Merchant)
CREATE (t)-[:FLAGGED_BY]->(rule:FraudRule)
```

### Generic Relationships

```cypher
// Anti-pattern: Generic relationship type
CREATE (a)-[:RELATED_TO {type: 'follows'}]->(b)
CREATE (a)-[:RELATED_TO {type: 'likes'}]->(c)
// Loses semantic meaning, harder to query

// Better: Specific relationship types
CREATE (a)-[:FOLLOWS]->(b)
CREATE (a)-[:LIKES]->(c)
// Query: MATCH ()-[:FOLLOWS]->() vs MATCH ()-[r:RELATED_TO {type:'follows'}]->()
```

### Dense Nodes (Supernodes)

```cypher
// Anti-pattern: Single node with millions of relationships
// (PopularCelebrity)-[:FOLLOWED_BY]->() × 10 million

// Solutions:
// 1. Fan-out structure
CREATE (celeb)-[:HAS_FOLLOWERS]->(shard:FollowerShard {number: 1})
CREATE (shard)-[:CONTAINS]->(follower)

// 2. Inverse relationship for counting
// Instead of counting all outgoing relationships
// Maintain counter property
SET celeb.follower_count = celeb.follower_count + 1

// 3. Time-based partitioning
CREATE (celeb)-[:FOLLOWERS_2024_01]->(shard)
```

### Missing Relationships

```cypher
// Anti-pattern: Implicit relationship via properties
CREATE (order:Order {customerId: 123})
// Query requires: MATCH (c:Customer), (o:Order) WHERE o.customerId = c.id

// Better: Explicit relationship
CREATE (customer:Customer {id: 123})-[:PLACED]->(order:Order)
// Query: MATCH (c:Customer)-[:PLACED]->(o:Order)
```

---

## 6. Schema and Constraints

### Indexes

```cypher
// Create index for fast lookup
CREATE INDEX person_name FOR (p:Person) ON (p.name)

// Composite index
CREATE INDEX person_name_age FOR (p:Person) ON (p.name, p.age)

// Text index (for full-text search)
CREATE TEXT INDEX person_bio FOR (p:Person) ON (p.bio)

// Relationship index
CREATE INDEX knows_since FOR ()-[k:KNOWS]-() ON (k.since)

// List indexes
SHOW INDEXES

// Drop index
DROP INDEX person_name
```

### Constraints

```cypher
// Unique constraint (also creates index)
CREATE CONSTRAINT unique_email FOR (p:Person)
REQUIRE p.email IS UNIQUE

// Node key (composite unique)
CREATE CONSTRAINT order_key FOR (o:Order)
REQUIRE (o.region, o.orderId) IS NODE KEY

// Existence constraint
CREATE CONSTRAINT person_name_exists FOR (p:Person)
REQUIRE p.name IS NOT NULL

// Relationship property existence
CREATE CONSTRAINT knows_since_exists FOR ()-[k:KNOWS]-()
REQUIRE k.since IS NOT NULL

// List constraints
SHOW CONSTRAINTS
```

---

## Summary

| Concept | Description |
|---------|-------------|
| Node | Entity with labels and properties |
| Relationship | Typed, directed connection with properties |
| Label | Category for nodes |
| Property | Key-value data on nodes/relationships |
| Pattern | Structural template for querying |

---

## Best Practices

```
Modeling:
✓ Draw on whiteboard first
✓ Use specific relationship types
✓ Make entities nodes if they connect to multiple things
✓ Use intermediate nodes for complex relationships

Naming:
✓ Labels: PascalCase (Person, OrderItem)
✓ Relationships: UPPER_SNAKE_CASE (PLACED_ORDER)
✓ Properties: camelCase (firstName, createdAt)

Performance:
✓ Create indexes for lookup properties
✓ Use constraints for data integrity
✓ Avoid supernodes (millions of relationships)
✓ Partition dense nodes if needed

Avoid:
✗ Generic relationship types
✗ Properties that should be relationships
✗ Over-normalized models
✗ Single-node bottlenecks
```
