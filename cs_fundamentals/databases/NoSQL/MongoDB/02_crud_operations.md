# CRUD Operations

## Learning Objectives
- Master insert, find, update, and delete operations
- Use query and update operators effectively
- Implement bulk operations for performance
- Configure write concerns and read preferences

---

## 1. Insert Operations

### insertOne

```javascript
// Insert single document
db.users.insertOne({
  name: "John Doe",
  email: "john@example.com",
  age: 30,
  createdAt: new Date()
})

// Returns:
{
  acknowledged: true,
  insertedId: ObjectId("507f1f77bcf86cd799439011")
}

// _id is auto-generated if not provided
db.users.insertOne({
  _id: "custom-id-123",  // Custom _id
  name: "Jane Doe"
})
```

### insertMany

```javascript
// Insert multiple documents
db.users.insertMany([
  { name: "Alice", age: 25 },
  { name: "Bob", age: 30 },
  { name: "Charlie", age: 35 }
])

// Returns:
{
  acknowledged: true,
  insertedIds: {
    '0': ObjectId("..."),
    '1': ObjectId("..."),
    '2': ObjectId("...")
  }
}

// Ordered insert (default: true)
// Stops on first error
db.users.insertMany([...], { ordered: true })

// Unordered insert
// Continues after errors, faster for large batches
db.users.insertMany([...], { ordered: false })
```

---

## 2. Find Operations

### Basic Queries

```javascript
// Find all documents
db.users.find()

// Find with filter
db.users.find({ age: 30 })

// Find one document
db.users.findOne({ email: "john@example.com" })

// Projection (select fields)
db.users.find(
  { age: { $gte: 25 } },        // Filter
  { name: 1, email: 1, _id: 0 } // Projection: include name, email; exclude _id
)

// Limit and skip
db.users.find().limit(10).skip(20)

// Sort
db.users.find().sort({ age: -1, name: 1 })  // age DESC, name ASC

// Count
db.users.countDocuments({ status: "active" })
db.users.estimatedDocumentCount()  // Fast, uses metadata
```

### Comparison Operators

```javascript
// $eq, $ne - Equal, Not Equal
db.users.find({ status: { $eq: "active" } })
db.users.find({ status: { $ne: "inactive" } })

// $gt, $gte, $lt, $lte - Greater/Less Than
db.users.find({ age: { $gt: 25 } })       // > 25
db.users.find({ age: { $gte: 25 } })      // >= 25
db.users.find({ age: { $lt: 30 } })       // < 30
db.users.find({ age: { $lte: 30 } })      // <= 30

// Range
db.users.find({ age: { $gte: 25, $lte: 35 } })

// $in, $nin - In Array, Not In Array
db.users.find({ status: { $in: ["active", "pending"] } })
db.users.find({ role: { $nin: ["admin", "superuser"] } })
```

### Logical Operators

```javascript
// $and (implicit when using multiple fields)
db.users.find({ status: "active", age: { $gte: 25 } })

// $and (explicit)
db.users.find({
  $and: [
    { age: { $gte: 25 } },
    { age: { $lte: 35 } }
  ]
})

// $or
db.users.find({
  $or: [
    { status: "active" },
    { role: "admin" }
  ]
})

// $not
db.users.find({
  age: { $not: { $gt: 30 } }  // NOT age > 30
})

// $nor (neither condition is true)
db.users.find({
  $nor: [
    { status: "inactive" },
    { deleted: true }
  ]
})

// Complex combination
db.users.find({
  $and: [
    { $or: [{ city: "NYC" }, { city: "LA" }] },
    { age: { $gte: 21 } }
  ]
})
```

### Element Operators

```javascript
// $exists - Field exists
db.users.find({ phone: { $exists: true } })
db.users.find({ deletedAt: { $exists: false } })

// $type - Field is specific type
db.users.find({ age: { $type: "int" } })
db.users.find({ age: { $type: ["int", "double"] } })
```

### Array Operators

```javascript
// Match array containing element
db.posts.find({ tags: "mongodb" })

// $all - Contains all elements
db.posts.find({ tags: { $all: ["mongodb", "database"] } })

// $size - Exact array length
db.posts.find({ tags: { $size: 3 } })

// $elemMatch - Element matches multiple conditions
db.orders.find({
  items: {
    $elemMatch: {
      product: "laptop",
      quantity: { $gte: 2 }
    }
  }
})

// Positional operator for projection
db.orders.find(
  { "items.product": "laptop" },
  { "items.$": 1 }  // Only matching array element
)
```

### Evaluation Operators

```javascript
// $regex - Regular expression
db.users.find({ name: { $regex: /^john/i } })
db.users.find({ email: { $regex: "example\\.com$" } })

// $expr - Aggregation expressions in find
db.orders.find({
  $expr: { $gt: ["$total", "$budget"] }  // Compare fields
})

// $text - Text search (requires text index)
db.articles.find({ $text: { $search: "mongodb tutorial" } })

// $where - JavaScript expression (slow, avoid if possible)
db.users.find({
  $where: function() { return this.age > this.minAge }
})
```

---

## 3. Update Operations

### updateOne and updateMany

```javascript
// Update one document
db.users.updateOne(
  { email: "john@example.com" },          // Filter
  { $set: { age: 31, updatedAt: new Date() } }  // Update
)

// Update many documents
db.users.updateMany(
  { status: "pending" },
  { $set: { status: "active" } }
)

// Returns:
{
  acknowledged: true,
  matchedCount: 1,
  modifiedCount: 1
}
```

### Update Operators

```javascript
// $set - Set field value
{ $set: { name: "John", "address.city": "NYC" } }

// $unset - Remove field
{ $unset: { temporaryField: "" } }

// $inc - Increment
{ $inc: { views: 1, score: -5 } }  // Negative for decrement

// $mul - Multiply
{ $mul: { price: 1.1 } }  // Increase by 10%

// $min, $max - Only update if new value is less/greater
{ $min: { lowScore: 50 } }   // Set if 50 < current
{ $max: { highScore: 100 } } // Set if 100 > current

// $rename - Rename field
{ $rename: { "old_name": "newName" } }

// $currentDate - Set to current date
{ $currentDate: { lastModified: true } }
{ $currentDate: { lastModified: { $type: "timestamp" } } }
```

### Array Update Operators

```javascript
// $push - Add to array
{ $push: { tags: "new-tag" } }

// $push with modifiers
{
  $push: {
    scores: {
      $each: [90, 92, 85],     // Multiple values
      $position: 0,            // Insert position
      $slice: -10,             // Keep last 10
      $sort: -1                // Sort descending
    }
  }
}

// $addToSet - Add only if not exists
{ $addToSet: { tags: "mongodb" } }
{ $addToSet: { tags: { $each: ["a", "b", "c"] } } }

// $pop - Remove first (-1) or last (1)
{ $pop: { scores: 1 } }   // Remove last
{ $pop: { scores: -1 } }  // Remove first

// $pull - Remove matching elements
{ $pull: { tags: "old-tag" } }
{ $pull: { items: { status: "cancelled" } } }

// $pullAll - Remove all matching values
{ $pullAll: { tags: ["a", "b", "c"] } }

// $ positional operator - Update matched element
db.orders.updateOne(
  { "items.product": "laptop" },
  { $set: { "items.$.quantity": 5 } }
)

// $[] - Update all array elements
db.orders.updateMany(
  {},
  { $inc: { "items.$[].quantity": 1 } }
)

// $[<identifier>] - Update filtered elements
db.orders.updateMany(
  {},
  { $set: { "items.$[elem].status": "shipped" } },
  { arrayFilters: [{ "elem.product": "laptop" }] }
)
```

### Upsert

```javascript
// Insert if not found, update if found
db.users.updateOne(
  { email: "new@example.com" },
  {
    $set: { name: "New User" },
    $setOnInsert: { createdAt: new Date() }  // Only on insert
  },
  { upsert: true }
)

// Returns:
{
  acknowledged: true,
  matchedCount: 0,
  modifiedCount: 0,
  upsertedId: ObjectId("...")  // When inserted
}
```

### findOneAndUpdate

```javascript
// Find, update, and return document
const result = db.users.findOneAndUpdate(
  { email: "john@example.com" },
  { $inc: { loginCount: 1 } },
  {
    returnDocument: "after",  // "before" or "after"
    projection: { name: 1, loginCount: 1 },
    upsert: false
  }
)

// Returns the document (or null if not found)
```

### replaceOne

```javascript
// Replace entire document (except _id)
db.users.replaceOne(
  { _id: ObjectId("...") },
  {
    name: "New Name",
    email: "new@example.com",
    // Complete replacement, no operators
  }
)
```

---

## 4. Delete Operations

### deleteOne and deleteMany

```javascript
// Delete one document
db.users.deleteOne({ email: "john@example.com" })

// Delete many documents
db.users.deleteMany({ status: "inactive" })

// Delete all documents
db.users.deleteMany({})

// Returns:
{
  acknowledged: true,
  deletedCount: 1
}
```

### findOneAndDelete

```javascript
// Find and delete, return deleted document
const deleted = db.queue.findOneAndDelete(
  { status: "pending" },
  { sort: { createdAt: 1 } }  // Delete oldest pending
)
// Returns the deleted document
```

---

## 5. Bulk Operations

### Bulk Write

```javascript
// Ordered bulk operations
db.collection.bulkWrite([
  { insertOne: { document: { name: "A" } } },
  { updateOne: {
    filter: { name: "B" },
    update: { $set: { status: "active" } }
  }},
  { updateMany: {
    filter: { status: "pending" },
    update: { $set: { status: "active" } }
  }},
  { deleteOne: { filter: { name: "C" } } },
  { deleteMany: { filter: { expired: true } } },
  { replaceOne: {
    filter: { name: "D" },
    replacement: { name: "D", status: "new" }
  }}
], { ordered: true })

// Unordered (parallel, faster)
db.collection.bulkWrite([...], { ordered: false })

// Returns:
{
  acknowledged: true,
  insertedCount: 1,
  matchedCount: 2,
  modifiedCount: 2,
  deletedCount: 5,
  upsertedCount: 0
}
```

### Performance Comparison

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Bulk vs Individual Operations                     │
│                                                                      │
│  Individual inserts (1000 documents):                                │
│  for (let i = 0; i < 1000; i++) {                                   │
│    db.collection.insertOne({ n: i })  // 1000 round trips          │
│  }                                                                   │
│  Time: ~5 seconds                                                    │
│                                                                      │
│  Bulk insert:                                                        │
│  db.collection.insertMany([...1000 docs])  // 1 round trip          │
│  Time: ~0.1 seconds (50x faster)                                     │
│                                                                      │
│  Best practice: Batch size of 100-1000 documents                    │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 6. Write Concern

### Write Concern Levels

```javascript
// Write concern controls acknowledgment
db.users.insertOne(
  { name: "John" },
  { writeConcern: { w: 1 } }  // Wait for primary
)

// Write concern options:
{
  w: 0,          // No acknowledgment (fire and forget)
  w: 1,          // Acknowledged by primary (default)
  w: "majority", // Acknowledged by majority of replica set
  w: 2,          // Acknowledged by 2 nodes
  j: true,       // Wait for journal write
  wtimeout: 5000 // Timeout in milliseconds
}

// Set at collection level
db.users.insertOne(
  { name: "Critical Data" },
  {
    writeConcern: {
      w: "majority",
      j: true,
      wtimeout: 10000
    }
  }
)
```

---

## 7. Read Preference

### Read Preference Modes

```javascript
// Read from primary (default)
db.users.find().readPref("primary")

// Read from secondary
db.users.find().readPref("secondary")

// Modes:
// primary: Only read from primary
// primaryPreferred: Primary, fallback to secondary
// secondary: Only read from secondary
// secondaryPreferred: Secondary, fallback to primary
// nearest: Lowest latency node

// With tags
db.users.find().readPref("secondary", [{ region: "us-east" }])

// Set at connection level
const client = new MongoClient(uri, {
  readPreference: "secondaryPreferred"
})
```

---

## 8. Practical Examples

### Pagination

```javascript
// Offset-based pagination (simple but slow for large offsets)
const page = 3
const pageSize = 20
db.products.find()
  .skip((page - 1) * pageSize)
  .limit(pageSize)
  .sort({ createdAt: -1 })

// Cursor-based pagination (efficient for large datasets)
const lastId = ObjectId("507f1f77bcf86cd799439011")
db.products.find({ _id: { $lt: lastId } })
  .sort({ _id: -1 })
  .limit(20)
```

### Atomic Counter

```javascript
// Get next sequence value
function getNextSequence(name) {
  const ret = db.counters.findOneAndUpdate(
    { _id: name },
    { $inc: { seq: 1 } },
    { returnDocument: "after", upsert: true }
  )
  return ret.seq
}

const nextOrderId = getNextSequence("orderId")
```

### Conditional Update

```javascript
// Update only if condition is met
db.inventory.updateOne(
  {
    product: "laptop",
    quantity: { $gte: 1 }  // Only if in stock
  },
  {
    $inc: { quantity: -1 },
    $push: { purchases: { date: new Date(), qty: 1 } }
  }
)
```

---

## Summary

| Operation | Method | Returns |
|-----------|--------|---------|
| Insert One | `insertOne()` | insertedId |
| Insert Many | `insertMany()` | insertedIds |
| Find | `find()` | Cursor |
| Find One | `findOne()` | Document |
| Update One | `updateOne()` | matchedCount, modifiedCount |
| Update Many | `updateMany()` | matchedCount, modifiedCount |
| Delete One | `deleteOne()` | deletedCount |
| Delete Many | `deleteMany()` | deletedCount |
| Bulk Write | `bulkWrite()` | Aggregate counts |

---

## Further Reading

- MongoDB CRUD Operations documentation
- Query and Projection Operators
- Update Operators Reference
