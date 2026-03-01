# Document Model

## 1. Introduction

The **document model** stores data as self-contained documents, typically in JSON or BSON format. Each document contains all the data for an entity, including nested structures and arrays, without requiring joins.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        DOCUMENT MODEL STRUCTURE                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Collection: users                                                          │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │ {                                                                    │   │
│   │   "_id": ObjectId("507f1f77bcf86cd799439011"),                      │   │
│   │   "name": "Alice Johnson",                                          │   │
│   │   "email": "alice@example.com",                                     │   │
│   │   "age": 28,                                                        │   │
│   │   "address": {                          ◄── Embedded document       │   │
│   │     "street": "123 Main St",                                        │   │
│   │     "city": "New York",                                             │   │
│   │     "zip": "10001"                                                  │   │
│   │   },                                                                │   │
│   │   "tags": ["developer", "python", "mongodb"],  ◄── Array           │   │
│   │   "orders": [                           ◄── Array of documents      │   │
│   │     { "order_id": "ORD001", "total": 99.99 },                       │   │
│   │     { "order_id": "ORD002", "total": 149.99 }                       │   │
│   │   ]                                                                 │   │
│   │ }                                                                   │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Core Concepts

### 2.1 Documents

A document is a self-describing, hierarchical data structure similar to JSON:

```json
{
    "_id": "user_12345",
    "username": "johndoe",
    "email": "john@example.com",
    "profile": {
        "firstName": "John",
        "lastName": "Doe",
        "bio": "Software engineer and coffee enthusiast",
        "avatar": "https://cdn.example.com/avatars/johndoe.jpg"
    },
    "preferences": {
        "theme": "dark",
        "language": "en-US",
        "notifications": {
            "email": true,
            "push": false
        }
    },
    "skills": ["JavaScript", "Python", "MongoDB", "Docker"],
    "employment": {
        "company": "Tech Corp",
        "position": "Senior Developer",
        "startDate": "2020-03-15"
    },
    "createdAt": "2019-06-01T10:30:00Z",
    "updatedAt": "2024-01-15T14:22:00Z"
}
```

### 2.2 Collections

Documents are organized into collections (analogous to tables):

```javascript
// MongoDB shell
db.createCollection("users")
db.createCollection("products")
db.createCollection("orders")

// Collections can have schema validation (optional)
db.createCollection("users", {
    validator: {
        $jsonSchema: {
            bsonType: "object",
            required: ["username", "email"],
            properties: {
                username: {
                    bsonType: "string",
                    description: "must be a string and is required"
                },
                email: {
                    bsonType: "string",
                    pattern: "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$"
                },
                age: {
                    bsonType: "int",
                    minimum: 0,
                    maximum: 150
                }
            }
        }
    }
})
```

### 2.3 Schema Flexibility

Documents in the same collection can have different structures:

```json
// Product 1: Electronics
{
    "_id": "prod_001",
    "name": "Gaming Laptop",
    "category": "Electronics",
    "price": 1299.99,
    "specs": {
        "cpu": "Intel i7-12700H",
        "ram": "16GB",
        "storage": "512GB SSD",
        "gpu": "RTX 3060"
    }
}

// Product 2: Clothing (different structure)
{
    "_id": "prod_002",
    "name": "Running Shoes",
    "category": "Clothing",
    "price": 129.99,
    "sizes": ["7", "8", "9", "10", "11"],
    "colors": ["black", "white", "red"],
    "material": "Mesh upper, rubber sole"
}

// Product 3: Book (completely different structure)
{
    "_id": "prod_003",
    "name": "Clean Code",
    "category": "Books",
    "price": 39.99,
    "author": "Robert C. Martin",
    "isbn": "978-0132350884",
    "pages": 464,
    "publisher": "Prentice Hall"
}
```

---

## 3. Data Modeling Patterns

### 3.1 Embedded Documents (Denormalization)

Store related data within the same document:

```json
// Blog post with embedded comments
{
    "_id": "post_001",
    "title": "Introduction to MongoDB",
    "content": "MongoDB is a document database...",
    "author": {
        "id": "user_123",
        "name": "Alice",
        "avatar": "https://cdn.example.com/alice.jpg"
    },
    "comments": [
        {
            "id": "comment_001",
            "author": { "id": "user_456", "name": "Bob" },
            "text": "Great article!",
            "createdAt": "2024-01-15T10:00:00Z",
            "likes": 5
        },
        {
            "id": "comment_002",
            "author": { "id": "user_789", "name": "Charlie" },
            "text": "Very helpful, thanks!",
            "createdAt": "2024-01-15T11:30:00Z",
            "likes": 3
        }
    ],
    "tags": ["mongodb", "database", "nosql"],
    "stats": {
        "views": 1500,
        "likes": 45,
        "shares": 12
    }
}
```

**When to embed:**
- One-to-one relationships
- One-to-few relationships (bounded arrays)
- Data that is always accessed together
- Data that doesn't change frequently

### 3.2 References (Normalization)

Store references to related documents:

```json
// User document
{
    "_id": "user_123",
    "name": "Alice",
    "email": "alice@example.com"
}

// Order document with reference
{
    "_id": "order_001",
    "user_id": "user_123",        // Reference to user
    "items": [
        {
            "product_id": "prod_001",  // Reference to product
            "quantity": 2,
            "price": 29.99
        }
    ],
    "total": 59.98,
    "status": "shipped"
}

// Lookup (join) in MongoDB
db.orders.aggregate([
    { $match: { _id: "order_001" } },
    {
        $lookup: {
            from: "users",
            localField: "user_id",
            foreignField: "_id",
            as: "user"
        }
    },
    { $unwind: "$user" }
])
```

**When to reference:**
- One-to-many with unbounded growth
- Many-to-many relationships
- Data that needs independent access
- Data that changes frequently

### 3.3 Hybrid Pattern

Combine embedding and referencing:

```json
// Order with denormalized user info + reference
{
    "_id": "order_001",
    "user": {
        "id": "user_123",           // Reference for full data
        "name": "Alice",            // Denormalized for display
        "email": "alice@example.com" // Denormalized for notifications
    },
    "items": [
        {
            "product_id": "prod_001",
            "name": "Gaming Laptop",  // Denormalized product name
            "price": 1299.99,
            "quantity": 1
        }
    ],
    "total": 1299.99
}
```

### 3.4 Bucket Pattern

Group related time-series data:

```json
// Temperature readings bucketed by hour
{
    "_id": "sensor_001_2024011510",  // sensor_date_hour
    "sensor_id": "sensor_001",
    "date": "2024-01-15",
    "hour": 10,
    "readings": [
        { "minute": 0, "temp": 22.5, "humidity": 45 },
        { "minute": 5, "temp": 22.6, "humidity": 44 },
        { "minute": 10, "temp": 22.4, "humidity": 46 },
        // ... up to 12 readings per hour (every 5 min)
    ],
    "summary": {
        "avgTemp": 22.5,
        "minTemp": 22.1,
        "maxTemp": 22.8,
        "count": 12
    }
}
```

### 3.5 Polymorphic Pattern

Single collection with varying document types:

```json
// Events collection with different event types
[
    {
        "_id": "evt_001",
        "type": "user_signup",
        "timestamp": "2024-01-15T10:00:00Z",
        "user_id": "user_123",
        "source": "web",
        "referrer": "google.com"
    },
    {
        "_id": "evt_002",
        "type": "purchase",
        "timestamp": "2024-01-15T10:05:00Z",
        "user_id": "user_123",
        "order_id": "order_001",
        "amount": 99.99,
        "payment_method": "credit_card"
    },
    {
        "_id": "evt_003",
        "type": "page_view",
        "timestamp": "2024-01-15T10:10:00Z",
        "user_id": "user_456",
        "page": "/products/laptop",
        "duration_seconds": 45
    }
]
```

---

## 4. CRUD Operations

### 4.1 Create

```javascript
// MongoDB Shell
// Insert one
db.users.insertOne({
    name: "Alice",
    email: "alice@example.com",
    createdAt: new Date()
})

// Insert many
db.users.insertMany([
    { name: "Bob", email: "bob@example.com" },
    { name: "Charlie", email: "charlie@example.com" }
])
```

### 4.2 Read

```javascript
// Find one
db.users.findOne({ email: "alice@example.com" })

// Find many with filter
db.users.find({ age: { $gte: 21 } })

// Projection (select fields)
db.users.find(
    { age: { $gte: 21 } },
    { name: 1, email: 1, _id: 0 }  // Include name, email; exclude _id
)

// Query nested documents
db.users.find({ "address.city": "New York" })

// Query arrays
db.users.find({ tags: "developer" })  // Contains "developer"
db.users.find({ tags: { $all: ["python", "mongodb"] } })  // Contains all

// Complex queries
db.products.find({
    $and: [
        { price: { $gte: 100, $lte: 500 } },
        { category: "Electronics" },
        { "specs.ram": { $regex: /16GB/ } }
    ]
}).sort({ price: -1 }).limit(10)
```

### 4.3 Update

```javascript
// Update one
db.users.updateOne(
    { email: "alice@example.com" },
    {
        $set: { name: "Alice Johnson" },
        $inc: { loginCount: 1 },
        $currentDate: { lastLogin: true }
    }
)

// Update many
db.products.updateMany(
    { category: "Electronics" },
    { $mul: { price: 1.1 } }  // 10% price increase
)

// Upsert (insert if not exists)
db.users.updateOne(
    { email: "new@example.com" },
    { $set: { name: "New User", createdAt: new Date() } },
    { upsert: true }
)

// Array operations
db.users.updateOne(
    { _id: "user_123" },
    {
        $push: { tags: "javascript" },           // Add to array
        $addToSet: { skills: "React" },          // Add if not exists
        $pull: { tags: "outdated" }              // Remove from array
    }
)

// Update nested array element
db.posts.updateOne(
    { _id: "post_001", "comments.id": "comment_001" },
    { $inc: { "comments.$.likes": 1 } }
)
```

### 4.4 Delete

```javascript
// Delete one
db.users.deleteOne({ email: "alice@example.com" })

// Delete many
db.sessions.deleteMany({ expiresAt: { $lt: new Date() } })

// Find and delete (returns deleted document)
db.queue.findOneAndDelete(
    { status: "pending" },
    { sort: { priority: -1 } }
)
```

---

## 5. Aggregation Pipeline

A powerful framework for data transformation and analysis:

```javascript
// Sales analytics pipeline
db.orders.aggregate([
    // Stage 1: Filter orders from 2024
    { $match: {
        orderDate: { $gte: ISODate("2024-01-01"), $lt: ISODate("2025-01-01") }
    }},

    // Stage 2: Unwind items array
    { $unwind: "$items" },

    // Stage 3: Lookup product details
    { $lookup: {
        from: "products",
        localField: "items.product_id",
        foreignField: "_id",
        as: "product"
    }},
    { $unwind: "$product" },

    // Stage 4: Group by category
    { $group: {
        _id: "$product.category",
        totalRevenue: { $sum: { $multiply: ["$items.quantity", "$items.price"] } },
        orderCount: { $sum: 1 },
        avgOrderValue: { $avg: { $multiply: ["$items.quantity", "$items.price"] } }
    }},

    // Stage 5: Sort by revenue
    { $sort: { totalRevenue: -1 } },

    // Stage 6: Format output
    { $project: {
        category: "$_id",
        totalRevenue: { $round: ["$totalRevenue", 2] },
        orderCount: 1,
        avgOrderValue: { $round: ["$avgOrderValue", 2] },
        _id: 0
    }}
])
```

### Common Aggregation Stages

```javascript
// $match - Filter documents
{ $match: { status: "active" } }

// $project - Reshape documents
{ $project: { name: 1, total: { $multiply: ["$price", "$quantity"] } } }

// $group - Group and aggregate
{ $group: { _id: "$category", count: { $sum: 1 }, avgPrice: { $avg: "$price" } } }

// $sort - Order results
{ $sort: { createdAt: -1 } }

// $limit / $skip - Pagination
{ $skip: 20 }
{ $limit: 10 }

// $unwind - Deconstruct arrays
{ $unwind: "$items" }

// $lookup - Join collections
{ $lookup: { from: "users", localField: "user_id", foreignField: "_id", as: "user" } }

// $addFields - Add computed fields
{ $addFields: { totalWithTax: { $multiply: ["$total", 1.08] } } }

// $facet - Multiple parallel pipelines
{ $facet: {
    byCategory: [{ $group: { _id: "$category", count: { $sum: 1 } } }],
    byMonth: [{ $group: { _id: { $month: "$date" }, count: { $sum: 1 } } }]
}}
```

---

## 6. Indexing

### 6.1 Index Types

```javascript
// Single field index
db.users.createIndex({ email: 1 })  // Ascending
db.users.createIndex({ createdAt: -1 })  // Descending

// Compound index
db.orders.createIndex({ user_id: 1, orderDate: -1 })

// Multikey index (for arrays)
db.products.createIndex({ tags: 1 })

// Text index (full-text search)
db.articles.createIndex({ title: "text", content: "text" })

// Geospatial index
db.locations.createIndex({ coordinates: "2dsphere" })

// Unique index
db.users.createIndex({ email: 1 }, { unique: true })

// Sparse index (only documents with field)
db.users.createIndex({ phone: 1 }, { sparse: true })

// TTL index (auto-delete after time)
db.sessions.createIndex({ createdAt: 1 }, { expireAfterSeconds: 3600 })

// Partial index (conditional)
db.orders.createIndex(
    { orderDate: 1 },
    { partialFilterExpression: { status: "active" } }
)
```

### 6.2 Index Strategies

```javascript
// Explain query execution
db.users.find({ email: "alice@example.com" }).explain("executionStats")

// Covered query (all fields in index)
db.users.createIndex({ email: 1, name: 1 })
db.users.find(
    { email: "alice@example.com" },
    { email: 1, name: 1, _id: 0 }  // Covered by index
)

// Index hints
db.orders.find({ user_id: "123" }).hint({ user_id: 1, orderDate: -1 })
```

---

## 7. Code Examples Across Languages

### Python (PyMongo)

```python
from pymongo import MongoClient
from bson.objectid import ObjectId
from datetime import datetime

# Connect
client = MongoClient('mongodb://localhost:27017/')
db = client['myapp']

# Insert
user = {
    'name': 'Alice',
    'email': 'alice@example.com',
    'profile': {
        'bio': 'Software developer',
        'skills': ['Python', 'MongoDB']
    },
    'createdAt': datetime.utcnow()
}
result = db.users.insert_one(user)
print(f"Inserted ID: {result.inserted_id}")

# Find
user = db.users.find_one({'email': 'alice@example.com'})
print(user['name'])

# Update
db.users.update_one(
    {'email': 'alice@example.com'},
    {
        '$set': {'profile.bio': 'Senior developer'},
        '$push': {'profile.skills': 'Docker'}
    }
)

# Aggregation
pipeline = [
    {'$match': {'profile.skills': 'Python'}},
    {'$group': {'_id': None, 'count': {'$sum': 1}}}
]
results = list(db.users.aggregate(pipeline))

# With context manager
with client.start_session() as session:
    with session.start_transaction():
        db.accounts.update_one(
            {'_id': 'acc1'},
            {'$inc': {'balance': -100}},
            session=session
        )
        db.accounts.update_one(
            {'_id': 'acc2'},
            {'$inc': {'balance': 100}},
            session=session
        )
```

### Java (MongoDB Driver)

```java
import com.mongodb.client.*;
import org.bson.Document;
import org.bson.types.ObjectId;
import java.util.*;

public class MongoExample {
    public static void main(String[] args) {
        MongoClient client = MongoClients.create("mongodb://localhost:27017");
        MongoDatabase db = client.getDatabase("myapp");
        MongoCollection<Document> users = db.getCollection("users");

        // Insert
        Document user = new Document()
            .append("name", "Alice")
            .append("email", "alice@example.com")
            .append("profile", new Document()
                .append("bio", "Software developer")
                .append("skills", Arrays.asList("Java", "MongoDB")))
            .append("createdAt", new Date());

        users.insertOne(user);

        // Find
        Document found = users.find(Filters.eq("email", "alice@example.com")).first();
        System.out.println(found.getString("name"));

        // Update
        users.updateOne(
            Filters.eq("email", "alice@example.com"),
            Updates.combine(
                Updates.set("profile.bio", "Senior developer"),
                Updates.push("profile.skills", "Docker")
            )
        );

        // Aggregation
        List<Document> pipeline = Arrays.asList(
            Aggregates.match(Filters.eq("profile.skills", "Java")),
            Aggregates.group(null, Accumulators.sum("count", 1))
        );
        users.aggregate(pipeline).forEach(doc -> System.out.println(doc));

        client.close();
    }
}
```

### JavaScript (Node.js)

```javascript
const { MongoClient, ObjectId } = require('mongodb');

async function main() {
    const client = new MongoClient('mongodb://localhost:27017');
    await client.connect();

    const db = client.db('myapp');
    const users = db.collection('users');

    // Insert
    const result = await users.insertOne({
        name: 'Alice',
        email: 'alice@example.com',
        profile: {
            bio: 'Software developer',
            skills: ['JavaScript', 'MongoDB']
        },
        createdAt: new Date()
    });
    console.log(`Inserted ID: ${result.insertedId}`);

    // Find
    const user = await users.findOne({ email: 'alice@example.com' });
    console.log(user.name);

    // Update
    await users.updateOne(
        { email: 'alice@example.com' },
        {
            $set: { 'profile.bio': 'Senior developer' },
            $push: { 'profile.skills': 'Docker' }
        }
    );

    // Aggregation
    const pipeline = [
        { $match: { 'profile.skills': 'JavaScript' } },
        { $group: { _id: null, count: { $sum: 1 } } }
    ];
    const results = await users.aggregate(pipeline).toArray();
    console.log(results);

    // Transaction
    const session = client.startSession();
    try {
        await session.withTransaction(async () => {
            await db.collection('accounts').updateOne(
                { _id: 'acc1' },
                { $inc: { balance: -100 } },
                { session }
            );
            await db.collection('accounts').updateOne(
                { _id: 'acc2' },
                { $inc: { balance: 100 } },
                { session }
            );
        });
    } finally {
        await session.endSession();
    }

    await client.close();
}

main().catch(console.error);
```

---

## 8. Advantages and Limitations

### Advantages

| Advantage | Description |
|-----------|-------------|
| **Flexible Schema** | Easily accommodate changing requirements |
| **Developer Friendly** | Natural mapping to application objects |
| **Horizontal Scaling** | Built-in sharding capabilities |
| **Performance** | Embedded data avoids joins |
| **Rich Queries** | Powerful aggregation framework |

### Limitations

| Limitation | Description |
|------------|-------------|
| **Data Duplication** | Denormalization leads to redundancy |
| **Transaction Scope** | Multi-document transactions have overhead |
| **Consistency** | Eventual consistency in distributed setups |
| **Memory Usage** | Documents must fit in memory for updates |
| **Joins** | $lookup is less efficient than SQL joins |

---

## 9. Summary

The document model excels when:
- Data has variable structure
- Performance requires avoiding joins
- Schema needs to evolve frequently
- Data naturally fits hierarchical structure
- Horizontal scaling is a priority

Choose documents over relational when the benefits of flexibility and performance outweigh the costs of data duplication and eventual consistency.
