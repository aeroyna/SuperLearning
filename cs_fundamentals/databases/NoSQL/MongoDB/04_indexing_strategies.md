# Indexing Strategies

## Learning Objectives
- Understand MongoDB index types
- Create and manage indexes effectively
- Optimize queries with proper indexing
- Monitor and analyze index usage

---

## 1. Index Fundamentals

### Why Indexes Matter

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Collection Scan vs Index Scan                     │
│                                                                      │
│  Without Index (Collection Scan):                                    │
│  ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐                          │
│  │ 1 │ 2 │ 3 │ 4 │ 5 │ 6 │ 7 │ 8 │ 9 │10 │  Scan all documents     │
│  └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘                          │
│  → Check every document: O(n)                                        │
│                                                                      │
│  With Index (Index Scan):                                            │
│  ┌─────────────────────────────────┐                                 │
│  │     B-Tree Index on "email"     │                                 │
│  │           ┌─────┐               │                                 │
│  │          ┌┤ M   ├┐              │                                 │
│  │         ┌┴┴─────┴┴┐             │                                 │
│  │        ┌┤ D │ R   ├┐            │                                 │
│  │       ─┴┴─────────┴┴─           │                                 │
│  └─────────────────────────────────┘                                 │
│  → Direct lookup: O(log n)                                           │
│                                                                      │
│  1 million documents:                                                │
│  Collection scan: ~1,000,000 comparisons                             │
│  Index scan: ~20 comparisons                                         │
└─────────────────────────────────────────────────────────────────────┘
```

### Creating Indexes

```javascript
// Single field index (ascending)
db.users.createIndex({ email: 1 })

// Single field index (descending)
db.users.createIndex({ createdAt: -1 })

// Unique index
db.users.createIndex({ email: 1 }, { unique: true })

// Compound index (multiple fields)
db.orders.createIndex({ customerId: 1, orderDate: -1 })

// Index with options
db.sessions.createIndex(
  { lastAccess: 1 },
  {
    name: "session_expiry_idx",
    expireAfterSeconds: 3600,  // TTL index
    background: true           // Don't block writes (deprecated in 4.2+)
  }
)

// List indexes
db.users.getIndexes()

// Drop index
db.users.dropIndex("email_1")
db.users.dropIndex({ email: 1 })

// Drop all indexes (except _id)
db.users.dropIndexes()
```

---

## 2. Index Types

### Single Field Index

```javascript
// Basic single field
db.products.createIndex({ sku: 1 })

// Queries that use this index:
db.products.find({ sku: "ABC123" })
db.products.find({ sku: { $in: ["A", "B", "C"] } })
db.products.find({ sku: { $gt: "A", $lt: "Z" } })
db.products.find().sort({ sku: 1 })   // Ascending sort
db.products.find().sort({ sku: -1 })  // Descending sort (works too)
```

### Compound Index

```javascript
// Multiple fields
db.orders.createIndex({ customerId: 1, orderDate: -1, status: 1 })

// Prefixes are usable:
db.orders.find({ customerId: 123 })                    // ✓ Uses index
db.orders.find({ customerId: 123, orderDate: "2024-01-01" }) // ✓ Uses index
db.orders.find({ customerId: 123, status: "shipped" }) // ✓ Partial, then scan

// Cannot skip prefix:
db.orders.find({ orderDate: "2024-01-01" })            // ✗ Cannot use index
db.orders.find({ status: "shipped" })                  // ✗ Cannot use index

// Sort must match index order
db.orders.find({ customerId: 123 })
  .sort({ orderDate: -1 })  // ✓ Matches index direction
  .sort({ orderDate: 1 })   // ✗ Opposite direction, may not use index
```

### Multikey Index (Arrays)

```javascript
// Index on array field
db.products.createIndex({ tags: 1 })

// Document: { tags: ["electronics", "sale", "featured"] }

// All queries use multikey index:
db.products.find({ tags: "electronics" })
db.products.find({ tags: { $in: ["sale", "clearance"] } })
db.products.find({ tags: { $all: ["electronics", "sale"] } })

// Limitations:
// - Cannot have compound multikey on multiple arrays
// - { a: [1,2], b: [3,4] } - Can only index one array field
```

### Text Index

```javascript
// Full-text search index
db.articles.createIndex({
  title: "text",
  content: "text",
  tags: "text"
}, {
  weights: {
    title: 10,    // Title matches score higher
    content: 5,
    tags: 1
  },
  name: "article_text_idx"
})

// Text search query
db.articles.find({
  $text: { $search: "mongodb database" }
})

// With score
db.articles.find(
  { $text: { $search: "mongodb" } },
  { score: { $meta: "textScore" } }
).sort({ score: { $meta: "textScore" } })

// Phrase search
db.articles.find({ $text: { $search: "\"exact phrase\"" } })

// Exclude terms
db.articles.find({ $text: { $search: "mongodb -mysql" } })

// Language-specific
db.articles.createIndex(
  { content: "text" },
  { default_language: "spanish" }
)
```

### Geospatial Indexes

```javascript
// 2dsphere index (for GeoJSON)
db.places.createIndex({ location: "2dsphere" })

// Document with GeoJSON point
db.places.insertOne({
  name: "Central Park",
  location: {
    type: "Point",
    coordinates: [-73.965355, 40.782865]  // [longitude, latitude]
  }
})

// Find near point
db.places.find({
  location: {
    $near: {
      $geometry: {
        type: "Point",
        coordinates: [-73.9667, 40.78]
      },
      $maxDistance: 1000  // meters
    }
  }
})

// Find within polygon
db.places.find({
  location: {
    $geoWithin: {
      $geometry: {
        type: "Polygon",
        coordinates: [[
          [-74, 40.7], [-74, 40.8], [-73.9, 40.8], [-73.9, 40.7], [-74, 40.7]
        ]]
      }
    }
  }
})
```

### Hashed Index

```javascript
// Hash-based index (for sharding)
db.users.createIndex({ shardKey: "hashed" })

// Properties:
// - Equality queries only (no range)
// - Even distribution for sharding
// - Cannot be unique or compound
```

---

## 3. Index Properties

### Unique Index

```javascript
// Unique constraint
db.users.createIndex({ email: 1 }, { unique: true })

// Compound unique
db.inventory.createIndex(
  { productId: 1, warehouseId: 1 },
  { unique: true }
)

// Partial unique (unique among subset)
db.users.createIndex(
  { email: 1 },
  {
    unique: true,
    partialFilterExpression: { email: { $exists: true } }
  }
)
```

### Sparse Index

```javascript
// Only index documents that have the field
db.users.createIndex(
  { phone: 1 },
  { sparse: true }
)

// Documents without phone field are NOT indexed
// Queries may not use sparse index if null/missing is relevant
```

### TTL Index (Time-To-Live)

```javascript
// Auto-delete documents after time
db.sessions.createIndex(
  { createdAt: 1 },
  { expireAfterSeconds: 3600 }  // Delete after 1 hour
)

// Documents deleted when createdAt + 3600s < now

// TTL on specific date field
db.events.createIndex(
  { expiresAt: 1 },
  { expireAfterSeconds: 0 }  // Delete at the specified time
)

// Document: { expiresAt: ISODate("2024-12-31T23:59:59Z") }
```

### Partial Index

```javascript
// Index only matching documents
db.orders.createIndex(
  { customerId: 1, orderDate: -1 },
  {
    partialFilterExpression: {
      status: { $in: ["pending", "processing"] }
    }
  }
)

// Smaller index, only includes active orders
// Queries must include filter condition to use index
db.orders.find({
  customerId: 123,
  status: "pending"  // Must include this to use partial index
})
```

---

## 4. Covered Queries

### What is a Covered Query?

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Covered Query                                     │
│                                                                      │
│  Query is "covered" when:                                            │
│  1. All query fields are in the index                               │
│  2. All projection fields are in the index                          │
│  3. No fields from document needed                                  │
│                                                                      │
│  Normal Query:                                                       │
│  Index → Get document IDs → Fetch documents → Return                │
│                                                                      │
│  Covered Query:                                                      │
│  Index → Return directly (no document fetch!)                       │
│                                                                      │
│  Much faster: No random I/O to fetch documents                      │
└─────────────────────────────────────────────────────────────────────┘
```

### Creating Covered Queries

```javascript
// Index
db.users.createIndex({ email: 1, name: 1, status: 1 })

// Covered query (all fields in index)
db.users.find(
  { email: "john@example.com" },
  { email: 1, name: 1, _id: 0 }  // Must exclude _id!
)

// Check if covered
db.users.find(
  { email: "john@example.com" },
  { email: 1, name: 1, _id: 0 }
).explain("executionStats")

// Look for: "totalDocsExamined": 0
// And: "stage": "IXSCAN" (not FETCH)
```

---

## 5. Query Analysis

### explain()

```javascript
// Basic explain
db.orders.find({ customerId: 123 }).explain()

// Execution statistics
db.orders.find({ customerId: 123 }).explain("executionStats")

// All plans considered
db.orders.find({ customerId: 123 }).explain("allPlansExecution")

// Key fields in explain output:
{
  "queryPlanner": {
    "winningPlan": {
      "stage": "FETCH",           // or IXSCAN, COLLSCAN
      "inputStage": {
        "stage": "IXSCAN",
        "indexName": "customerId_1",
        "direction": "forward"
      }
    }
  },
  "executionStats": {
    "nReturned": 10,              // Documents returned
    "executionTimeMillis": 5,     // Time taken
    "totalKeysExamined": 10,      // Index entries scanned
    "totalDocsExamined": 10       // Documents scanned
  }
}
```

### Common Stages

```
COLLSCAN: Full collection scan (no index)
IXSCAN: Index scan
FETCH: Retrieve documents
SORT: In-memory sort (not using index)
SORT_KEY_GENERATOR: Prepare for sort
PROJECTION: Apply projection
LIMIT: Limit results
SKIP: Skip results
```

### Index Usage Metrics

```javascript
// Index usage statistics
db.users.aggregate([
  { $indexStats: {} }
])

// Returns:
{
  "name": "email_1",
  "key": { "email": 1 },
  "accesses": {
    "ops": 12345,        // Times used
    "since": ISODate()   // Since when
  }
}

// Find unused indexes
db.users.aggregate([
  { $indexStats: {} },
  { $match: { "accesses.ops": 0 } }
])
```

---

## 6. Index Selection Guidelines

### ESR Rule (Equality, Sort, Range)

```javascript
// For compound indexes, order fields by:
// 1. Equality (exact match fields)
// 2. Sort (order by fields)
// 3. Range (range query fields)

// Query:
db.orders.find({
  customerId: 123,           // Equality
  amount: { $gte: 100 }      // Range
}).sort({ orderDate: -1 })   // Sort

// Best index:
db.orders.createIndex({
  customerId: 1,             // Equality first
  orderDate: -1,             // Sort second
  amount: 1                  // Range last
})
```

### Common Patterns

```javascript
// Pattern 1: Filter + Sort
// Query: find by status, sort by date
db.orders.createIndex({ status: 1, createdAt: -1 })

// Pattern 2: Prefix lookups
// Query: find by user, then by various criteria
db.orders.createIndex({ userId: 1, status: 1, createdAt: -1 })

// Pattern 3: Range queries
// Query: find in date range
db.events.createIndex({ eventType: 1, timestamp: 1 })

// Pattern 4: Covered queries
// Query: return specific fields only
db.products.createIndex({ sku: 1, name: 1, price: 1 })
```

---

## 7. Index Management

### Monitoring

```javascript
// Current index operations
db.currentOp({ "command.createIndexes": { $exists: true } })

// Index build progress
db.adminCommand({ currentOp: true, command: { createIndexes: { $exists: true } } })

// Server status
db.serverStatus().indexBuilds
```

### Maintenance

```javascript
// Rebuild indexes (rarely needed)
db.collection.reIndex()

// Compact collection (recover space)
db.runCommand({ compact: "collection" })

// Validate index integrity
db.collection.validate({ full: true })
```

### Index Hints

```javascript
// Force specific index
db.orders.find({ customerId: 123, status: "pending" })
  .hint({ customerId: 1, status: 1 })

// Force collection scan (for comparison)
db.orders.find({ customerId: 123 })
  .hint({ $natural: 1 })
```

---

## 8. Best Practices

### Do's

```
✓ Create indexes for common query patterns
✓ Use compound indexes to cover multiple queries
✓ Put selective fields first in compound indexes
✓ Use partial indexes for subset of documents
✓ Monitor index usage and remove unused indexes
✓ Align index direction with sort direction
✓ Include projection fields for covered queries
```

### Don'ts

```
✗ Don't create indexes for every field
✗ Don't create redundant indexes
✗ Don't ignore index size (RAM impact)
✗ Don't forget about write performance impact
✗ Don't use highly selective field last in compound
✗ Don't create indexes during peak hours
```

---

## Summary

| Index Type | Use Case |
|------------|----------|
| Single Field | Basic queries on one field |
| Compound | Multi-field queries, sorting |
| Multikey | Array fields |
| Text | Full-text search |
| Geospatial | Location queries |
| Hashed | Sharding, equality only |
| TTL | Auto-expiring documents |

---

## Further Reading

- MongoDB Index documentation
- Index Strategies
- Analyze Query Performance
