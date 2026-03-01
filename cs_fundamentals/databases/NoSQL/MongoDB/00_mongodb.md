# MongoDB

## Overview

MongoDB is the leading document-oriented NoSQL database, designed for flexibility, scalability, and developer productivity. It stores data in flexible, JSON-like documents, making it ideal for applications with evolving schemas and complex data structures.

---

## What You'll Learn

### 1. Document Model and BSON
- JSON and BSON fundamentals
- Document structure and embedded documents
- ObjectId and data types
- Schema flexibility vs validation

### 2. CRUD Operations
- Insert, find, update, delete operations
- Query operators and projections
- Bulk operations
- Write concerns and read preferences

### 3. Aggregation Pipeline
- Pipeline stages and operators
- Data transformation and analysis
- Window functions
- Real-time analytics patterns

### 4. Indexing Strategies
- Index types (single, compound, multikey)
- Text and geospatial indexes
- Index optimization
- Covered queries

### 5. Sharding and Replication
- Replica sets for high availability
- Horizontal scaling with sharding
- Shard key selection
- Zone sharding

### 6. Schema Design Patterns
- Embedding vs referencing
- Common design patterns
- Anti-patterns to avoid
- Migration strategies

---

## Why MongoDB?

```
┌─────────────────────────────────────────────────────────────────────┐
│                    MongoDB Value Proposition                         │
│                                                                      │
│  DOCUMENT MODEL                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  • Natural mapping to objects in code                        │    │
│  │  • Embedded documents reduce joins                           │    │
│  │  • Schema flexibility for rapid development                  │    │
│  │  • Rich query language                                       │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  SCALABILITY                                                         │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  • Automatic sharding for horizontal scale                   │    │
│  │  • Built-in replication for high availability                │    │
│  │  • Global clusters for worldwide deployment                  │    │
│  │  • Handle billions of documents                              │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  DEVELOPER EXPERIENCE                                                │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  • Native drivers for all major languages                   │    │
│  │  • Powerful aggregation framework                           │    │
│  │  • Change streams for real-time                              │    │
│  │  • Full-text search (Atlas Search)                          │    │
│  └─────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
```

---

## MongoDB vs Relational Databases

```
┌───────────────────────────────────────────────────────────────────────┐
│                    MongoDB vs RDBMS Concepts                          │
│                                                                        │
│  RDBMS              MongoDB            Notes                           │
│  ─────────────────────────────────────────────────────────────────    │
│  Database           Database           Same concept                    │
│  Table              Collection         Group of documents              │
│  Row                Document           JSON-like object                │
│  Column             Field              Document attribute              │
│  Primary Key        _id                Auto-generated ObjectId        │
│  JOIN               $lookup /          Embedding preferred            │
│                     Embedding                                          │
│  Index              Index              Similar concepts                │
│  Transaction        Transaction        Multi-doc since 4.0            │
│                                                                        │
│  Key Differences:                                                      │
│  • No enforced schema (schema-less or schema-flexible)                │
│  • Documents can have different structures in same collection         │
│  • Nested documents replace many-to-one relationships                 │
│  • Arrays can store multiple values per field                         │
└───────────────────────────────────────────────────────────────────────┘
```

---

## Document Example

```javascript
// A typical MongoDB document
{
  "_id": ObjectId("507f1f77bcf86cd799439011"),
  "name": "John Doe",
  "email": "john@example.com",
  "age": 30,
  "address": {                          // Embedded document
    "street": "123 Main St",
    "city": "New York",
    "country": "USA"
  },
  "tags": ["developer", "mongodb"],     // Array
  "orders": [                           // Array of embedded documents
    {
      "product": "Laptop",
      "quantity": 1,
      "price": 999.99,
      "date": ISODate("2024-01-15")
    },
    {
      "product": "Mouse",
      "quantity": 2,
      "price": 49.99,
      "date": ISODate("2024-01-20")
    }
  ],
  "createdAt": ISODate("2024-01-01"),
  "updatedAt": ISODate("2024-01-20"),
  "active": true
}
```

---

## Quick Reference

```javascript
// Connect
mongosh "mongodb://localhost:27017/mydb"

// Basic CRUD
db.users.insertOne({ name: "John", age: 30 })
db.users.find({ age: { $gte: 25 } })
db.users.updateOne({ name: "John" }, { $set: { age: 31 } })
db.users.deleteOne({ name: "John" })

// Aggregation
db.orders.aggregate([
  { $match: { status: "completed" } },
  { $group: { _id: "$customerId", total: { $sum: "$amount" } } },
  { $sort: { total: -1 } }
])

// Indexes
db.users.createIndex({ email: 1 }, { unique: true })
db.users.getIndexes()
```

---

## Section Files

| File | Topic |
|------|-------|
| [01_document_model_and_bson.md](01_document_model_and_bson.md) | BSON, documents, data types |
| [02_crud_operations.md](02_crud_operations.md) | Create, read, update, delete |
| [03_aggregation_pipeline.md](03_aggregation_pipeline.md) | Data processing pipelines |
| [04_indexing_strategies.md](04_indexing_strategies.md) | Index types and optimization |
| [05_sharding_and_replication.md](05_sharding_and_replication.md) | Scaling and HA |
| [06_schema_design_patterns.md](06_schema_design_patterns.md) | Design best practices |

---

## Use Cases

```
When to Choose MongoDB:

✓ Rapidly evolving schema requirements
✓ Content management systems
✓ Product catalogs with varying attributes
✓ Real-time analytics and logging
✓ IoT and time-series data
✓ Mobile application backends
✓ Caching layer with persistence

When to Consider Alternatives:

✗ Complex transactions across many tables
✗ Heavy JOIN requirements
✗ Strict schema enforcement critical
✗ Simple key-value needs (use Redis)
✗ Graph relationships (use Neo4j)
```

---

## Further Reading

- MongoDB Manual
- MongoDB University (free courses)
- "MongoDB: The Definitive Guide" by Kristina Chodorow
