# Schema Design Patterns

## Learning Objectives
- Master embedding vs referencing decisions
- Apply common MongoDB design patterns
- Avoid anti-patterns that hurt performance
- Design schemas for specific use cases

---

## 1. Embedding vs Referencing

### When to Embed

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Embedding (Denormalization)                       │
│                                                                      │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │ {                                                               │ │
│  │   "_id": ObjectId("..."),                                       │ │
│  │   "name": "John Doe",                                           │ │
│  │   "address": {                    ← Embedded document           │ │
│  │     "street": "123 Main St",                                    │ │
│  │     "city": "New York"                                          │ │
│  │   },                                                            │ │
│  │   "orders": [                     ← Embedded array              │ │
│  │     { "product": "A", "qty": 2 },                               │ │
│  │     { "product": "B", "qty": 1 }                                │ │
│  │   ]                                                             │ │
│  │ }                                                               │ │
│  └────────────────────────────────────────────────────────────────┘ │
│                                                                      │
│  Embed when:                                                         │
│  • One-to-one relationship                                          │
│  • One-to-few relationship (bounded array)                          │
│  • Data is always accessed together                                 │
│  • Data doesn't change frequently                                   │
│  • Child data doesn't need independent access                       │
└─────────────────────────────────────────────────────────────────────┘
```

### When to Reference

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Referencing (Normalization)                       │
│                                                                      │
│  Users Collection:                    Orders Collection:            │
│  ┌─────────────────────┐              ┌─────────────────────┐       │
│  │ {                   │              │ {                   │       │
│  │   "_id": 1,         │◀─reference──│   "userId": 1,      │       │
│  │   "name": "John"    │              │   "product": "A"    │       │
│  │ }                   │              │ }                   │       │
│  └─────────────────────┘              └─────────────────────┘       │
│                                                                      │
│  Reference when:                                                     │
│  • One-to-many (unbounded)                                          │
│  • Many-to-many relationships                                       │
│  • Data accessed independently                                      │
│  • Data changes frequently                                          │
│  • Document would exceed 16MB                                       │
│  • Need to query child data independently                          │
└─────────────────────────────────────────────────────────────────────┘
```

### Hybrid Approach

```javascript
// Best of both worlds
{
  "_id": ObjectId("..."),
  "name": "John Doe",

  // Embed frequently accessed summary
  "recentOrders": [
    { "orderId": 1001, "date": "2024-01-15", "total": 99.99 },
    { "orderId": 1002, "date": "2024-01-20", "total": 149.99 }
  ],

  // Reference for full details
  "orderCount": 150,
  "totalSpent": 15000.00
}

// Full order details in separate collection
// Orders collection has userId for lookups
```

---

## 2. Common Patterns

### Attribute Pattern

```javascript
// Problem: Products with varying attributes
// Anti-pattern: Sparse fields
{
  "name": "T-Shirt",
  "size": "L",           // Clothing only
  "color": "blue",       // Clothing only
  "screenSize": null,    // Electronics only
  "resolution": null     // Electronics only
}

// Attribute Pattern: Use array of key-value pairs
{
  "name": "T-Shirt",
  "category": "clothing",
  "attributes": [
    { "k": "size", "v": "L" },
    { "k": "color", "v": "blue" },
    { "k": "material", "v": "cotton" }
  ]
}

// Index for attribute queries
db.products.createIndex({ "attributes.k": 1, "attributes.v": 1 })

// Query
db.products.find({
  "attributes": { $elemMatch: { k: "color", v: "blue" } }
})
```

### Bucket Pattern

```javascript
// Problem: Time-series data with many small documents
// Anti-pattern: One document per measurement
{ "sensor": "A", "time": ISODate("..."), "value": 42 }

// Bucket Pattern: Group measurements
{
  "sensor": "A",
  "date": ISODate("2024-01-15"),
  "measurements": [
    { "time": ISODate("2024-01-15T00:00:00"), "value": 42 },
    { "time": ISODate("2024-01-15T00:01:00"), "value": 43 },
    { "time": ISODate("2024-01-15T00:02:00"), "value": 41 }
    // ... more measurements
  ],
  "count": 1440,  // Measurements in bucket
  "sum": 60480,   // Pre-computed for averages
  "min": 38,
  "max": 47
}

// Benefits:
// - Fewer documents (better performance)
// - Pre-aggregated statistics
// - Reduced index size
```

### Extended Reference Pattern

```javascript
// Problem: Need some data from referenced collection
// Anti-pattern: Always $lookup for any display

// Extended Reference: Copy frequently needed fields
{
  "_id": ObjectId("..."),
  "customerId": ObjectId("cust123"),

  // Extended reference - copied fields
  "customerName": "John Doe",
  "customerEmail": "john@example.com",

  "items": [...],
  "total": 299.99
}

// Update customer name? Update in orders too
// Use change streams or application logic
db.customers.watch().on("change", (change) => {
  if (change.operationType === "update" && change.updateDescription.updatedFields.name) {
    db.orders.updateMany(
      { customerId: change.documentKey._id },
      { $set: { customerName: change.updateDescription.updatedFields.name } }
    )
  }
})
```

### Subset Pattern

```javascript
// Problem: Large arrays slow down queries
// Anti-pattern: Embed all reviews in product

// Subset Pattern: Embed recent subset, reference rest
{
  "_id": ObjectId("..."),
  "name": "Awesome Product",
  "price": 99.99,

  // Subset of recent reviews (always loaded)
  "recentReviews": [
    { "user": "Alice", "rating": 5, "date": "2024-01-20" },
    { "user": "Bob", "rating": 4, "date": "2024-01-19" }
  ],

  // Summary statistics
  "reviewCount": 1250,
  "averageRating": 4.5
}

// Full reviews in separate collection
{
  "_id": ObjectId("..."),
  "productId": ObjectId("prod123"),
  "user": "Charlie",
  "rating": 5,
  "comment": "Great product!",
  "date": ISODate("2024-01-18")
}
```

### Computed Pattern

```javascript
// Problem: Expensive aggregations on every read
// Anti-pattern: Calculate on every query

// Computed Pattern: Pre-calculate and store
{
  "_id": ObjectId("..."),
  "name": "Product A",
  "price": 99.99,

  // Computed fields (updated periodically or on write)
  "stats": {
    "totalSales": 1500,
    "revenue": 149985.00,
    "avgRating": 4.7,
    "lastCalculated": ISODate("2024-01-15T10:00:00Z")
  }
}

// Update on relevant events
db.products.updateOne(
  { _id: productId },
  {
    $inc: { "stats.totalSales": 1, "stats.revenue": 99.99 },
    $set: { "stats.lastCalculated": new Date() }
  }
)
```

### Outlier Pattern

```javascript
// Problem: Some documents are much larger than others
// Example: Most users have 0-10 orders, one has 100,000

// Outlier Pattern: Handle outliers differently
{
  "_id": ObjectId("..."),
  "name": "Regular User",
  "orders": [/* 5 orders embedded */]
}

{
  "_id": ObjectId("..."),
  "name": "Power User",
  "hasOverflow": true,  // Flag for outlier
  "orders": [/* last 10 orders */],
  "orderCount": 100000
}

// Query with overflow handling
async function getUserOrders(userId) {
  const user = await db.users.findOne({ _id: userId })

  if (user.hasOverflow) {
    // Fetch from overflow collection
    return db.user_orders.find({ userId }).toArray()
  }

  return user.orders
}
```

### Schema Versioning Pattern

```javascript
// Problem: Schema evolves over time
// Need to handle old and new formats

// Version field in documents
{
  "_id": ObjectId("..."),
  "schemaVersion": 2,
  "name": "John Doe",
  "contacts": {  // v2 structure
    "email": "john@example.com",
    "phone": "+1-555-123-4567"
  }
}

// Migration function
function migrateUser(doc) {
  if (doc.schemaVersion === 1) {
    return {
      ...doc,
      schemaVersion: 2,
      contacts: {
        email: doc.email,
        phone: doc.phone
      }
    }
  }
  return doc
}

// Lazy migration on read
db.users.find().forEach(doc => {
  const migrated = migrateUser(doc)
  if (migrated.schemaVersion !== doc.schemaVersion) {
    db.users.replaceOne({ _id: doc._id }, migrated)
  }
})
```

---

## 3. Anti-Patterns

### Unbounded Arrays

```javascript
// BAD: Array grows without limit
{
  "userId": 1,
  "followers": [/* could be millions */]
}

// GOOD: Separate collection with pagination
// Followers collection
{ "userId": 1, "followerId": 100 }
{ "userId": 1, "followerId": 101 }

// Or bucket pattern for counts
{
  "userId": 1,
  "followerCount": 1500000,
  "recentFollowers": [/* last 100 */]
}
```

### Massive Documents

```javascript
// BAD: Single document with everything
{
  "user": "John",
  "allPosts": [/* 10,000 posts */],
  "allComments": [/* 50,000 comments */],
  "allLikes": [/* 100,000 likes */]
}
// Exceeds 16MB, slow to load, hard to update

// GOOD: Separate collections
// Users: { _id, name, postCount }
// Posts: { _id, userId, content }
// Comments: { _id, postId, content }
```

### Over-Normalization

```javascript
// BAD: Relational thinking in MongoDB
// Users: { _id, name, addressId }
// Addresses: { _id, street, city }
// Requires $lookup for every user query

// GOOD: Embed when data is owned
{
  "_id": ObjectId("..."),
  "name": "John",
  "address": {
    "street": "123 Main St",
    "city": "NYC"
  }
}
```

### Case-Sensitive Queries Without Index

```javascript
// BAD: Case-insensitive regex on every query
db.users.find({ email: /^john@example\.com$/i })
// Full collection scan!

// GOOD: Store normalized value
{
  "email": "John@Example.com",
  "emailLower": "john@example.com"  // Indexed
}

// Or use collation
db.users.createIndex(
  { email: 1 },
  { collation: { locale: "en", strength: 2 } }
)
```

---

## 4. Use Case Examples

### E-Commerce Product Catalog

```javascript
// Product with variants
{
  "_id": ObjectId("..."),
  "name": "Premium T-Shirt",
  "brand": "BrandX",
  "category": ["clothing", "tops", "t-shirts"],
  "basePrice": 29.99,

  // Variants embedded (bounded)
  "variants": [
    {
      "sku": "TS-BL-S",
      "color": "blue",
      "size": "S",
      "price": 29.99,
      "inventory": 50
    },
    {
      "sku": "TS-BL-M",
      "color": "blue",
      "size": "M",
      "price": 29.99,
      "inventory": 75
    }
  ],

  // Review summary (computed)
  "reviewStats": {
    "count": 150,
    "average": 4.5
  },

  // SEO and search
  "searchKeywords": ["tshirt", "t-shirt", "tee", "casual"],

  "createdAt": ISODate("..."),
  "updatedAt": ISODate("...")
}

// Indexes
db.products.createIndex({ "category": 1, "basePrice": 1 })
db.products.createIndex({ "variants.sku": 1 }, { unique: true })
db.products.createIndex({ "searchKeywords": 1 })
```

### Social Media Feed

```javascript
// Post document
{
  "_id": ObjectId("..."),
  "authorId": ObjectId("..."),

  // Extended reference
  "author": {
    "name": "John Doe",
    "avatar": "https://..."
  },

  "content": "Hello world!",
  "media": [
    { "type": "image", "url": "https://..." }
  ],

  // Engagement counters
  "likes": 42,
  "comments": 15,
  "shares": 5,

  // Recent comments (subset)
  "recentComments": [
    {
      "authorId": ObjectId("..."),
      "authorName": "Jane",
      "text": "Great post!",
      "createdAt": ISODate("...")
    }
  ],

  "createdAt": ISODate("..."),
  "visibility": "public"
}

// Separate collection for all comments
{
  "_id": ObjectId("..."),
  "postId": ObjectId("..."),
  "authorId": ObjectId("..."),
  "text": "Nice!",
  "createdAt": ISODate("...")
}
```

---

## Summary

| Pattern | Use Case |
|---------|----------|
| Attribute | Variable product attributes |
| Bucket | Time-series, IoT data |
| Extended Reference | Reduce lookups |
| Subset | Large arrays with recent subset |
| Computed | Pre-calculated aggregations |
| Outlier | Handle exceptional documents |
| Schema Versioning | Evolving schemas |

---

## Further Reading

- MongoDB Schema Design Anti-Patterns
- Building with Patterns blog series
- MongoDB University M320: Data Modeling
