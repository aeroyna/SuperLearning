# Document Model and BSON

## Learning Objectives
- Understand BSON format and data types
- Master document structure and nesting
- Work with ObjectId and special types
- Implement schema validation

---

## 1. BSON Fundamentals

### What is BSON?

```
┌─────────────────────────────────────────────────────────────────────┐
│                    BSON (Binary JSON)                                │
│                                                                      │
│  JSON:                              BSON:                           │
│  ┌────────────────────────┐        ┌────────────────────────┐       │
│  │ Text format            │        │ Binary format          │       │
│  │ Human readable         │        │ Machine optimized      │       │
│  │ Limited types          │        │ Extended types         │       │
│  │ No size prefix         │        │ Size prefix (fast seek)│       │
│  └────────────────────────┘        └────────────────────────┘       │
│                                                                      │
│  BSON Advantages:                                                    │
│  • Richer type system (Date, Binary, ObjectId, etc.)                │
│  • Efficient encoding/decoding                                      │
│  • Traversable (can skip fields without parsing)                    │
│  • Designed for MongoDB's storage and wire protocol                 │
│                                                                      │
│  Size comparison (typical document):                                 │
│  JSON:  {"name":"John","age":30}        20 bytes (text)             │
│  BSON:  \x1E\x00\x00\x00...             30 bytes (binary + metadata)│
│  BSON is slightly larger but much faster to process                 │
└─────────────────────────────────────────────────────────────────────┘
```

### BSON Data Types

```javascript
// All BSON data types
{
  // Basic types
  string: "Hello World",                    // String (UTF-8)
  int32: NumberInt(42),                     // 32-bit integer
  int64: NumberLong(9007199254740993),      // 64-bit integer
  double: 3.14159,                          // 64-bit floating point
  boolean: true,                            // Boolean
  null: null,                               // Null value

  // Object types
  objectId: ObjectId("507f1f77bcf86cd799439011"),  // 12-byte unique ID
  date: ISODate("2024-01-15T10:30:00Z"),           // UTC datetime
  timestamp: Timestamp(1705316400, 1),             // Internal timestamp

  // Binary types
  binData: BinData(0, "SGVsbG8gV29ybGQ="),  // Binary data (Base64)
  uuid: UUID("550e8400-e29b-41d4-a716-446655440000"),

  // Special types
  regex: /pattern/i,                        // Regular expression
  javascript: function() { return 1; },     // JavaScript code (deprecated)
  minKey: MinKey(),                         // Lowest possible value
  maxKey: MaxKey(),                         // Highest possible value

  // Decimal (MongoDB 3.4+)
  decimal: NumberDecimal("9.99"),           // 128-bit decimal

  // Nested types
  document: { nested: "value" },            // Embedded document
  array: [1, 2, 3, "mixed", { obj: true }]  // Array (mixed types OK)
}
```

### Type Checking

```javascript
// Check document types
db.collection.find({
  field: { $type: "string" }      // By type name
})

db.collection.find({
  field: { $type: 2 }             // By BSON type number
})

// Type numbers:
// 1: Double       7: ObjectId    13: JavaScript
// 2: String       8: Boolean     14: Symbol
// 3: Object       9: Date        16: 32-bit int
// 4: Array       10: Null        17: Timestamp
// 5: Binary      11: Regex       18: 64-bit int
// 6: Undefined   12: DBPointer   19: Decimal128

// Check for multiple types
db.collection.find({
  field: { $type: ["string", "null"] }
})
```

---

## 2. ObjectId

### ObjectId Structure

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ObjectId Structure (12 bytes)                     │
│                                                                      │
│  507f1f77 bcf86cd799 439011                                         │
│  ├──────┤ ├────────┤ ├────┤                                         │
│     │         │        │                                             │
│     │         │        └── Counter (3 bytes)                         │
│     │         │            Starts at random, increments              │
│     │         │                                                      │
│     │         └── Random value (5 bytes)                             │
│     │             Unique per machine/process                         │
│     │                                                                │
│     └── Timestamp (4 bytes)                                          │
│         Seconds since Unix epoch                                     │
│                                                                      │
│  Properties:                                                         │
│  • Globally unique without coordination                              │
│  • Roughly sorted by creation time                                   │
│  • Can extract timestamp from _id                                    │
│  • 12 bytes = 24 hex characters                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Working with ObjectId

```javascript
// Generate new ObjectId
const id = new ObjectId()
// ObjectId("507f1f77bcf86cd799439011")

// Create from string
const id2 = ObjectId("507f1f77bcf86cd799439011")

// Extract timestamp
id.getTimestamp()
// ISODate("2012-10-17T20:46:22Z")

// Query by ObjectId
db.users.find({ _id: ObjectId("507f1f77bcf86cd799439011") })

// Query by time range using _id
const startOfDay = ObjectId.fromDate(new Date("2024-01-15"))
db.collection.find({
  _id: { $gte: startOfDay }
})

// Compare ObjectIds
id1.equals(id2)  // Boolean comparison
id1.toString()   // "507f1f77bcf86cd799439011"
```

---

## 3. Document Structure

### Basic Document

```javascript
// Document is a JSON-like object with BSON types
{
  "_id": ObjectId("..."),    // Required, auto-generated if omitted
  "field1": "value1",
  "field2": 123,
  "nested": {
    "subfield": "subvalue"
  }
}

// Field name rules:
// • Cannot start with $ (reserved for operators)
// • Cannot contain null character
// • Cannot contain . (dot) in field names
// • _id is reserved for primary key
// • Top-level _id cannot be an array
```

### Embedded Documents

```javascript
// Embedding related data (denormalization)
{
  "_id": ObjectId("..."),
  "name": "John Doe",
  "address": {                      // Embedded document
    "street": "123 Main St",
    "city": "New York",
    "state": "NY",
    "zip": "10001",
    "geo": {                        // Nested embedding
      "type": "Point",
      "coordinates": [-73.935242, 40.730610]
    }
  },
  "contacts": {
    "email": "john@example.com",
    "phone": "+1-555-123-4567"
  }
}

// Query embedded fields with dot notation
db.users.find({ "address.city": "New York" })
db.users.find({ "address.geo.coordinates": { $near: [-73.9, 40.7] } })
```

### Arrays

```javascript
// Arrays can contain any BSON type
{
  "tags": ["mongodb", "database", "nosql"],           // Array of strings
  "scores": [95, 87, 92, 88],                         // Array of numbers
  "items": [                                           // Array of documents
    { "product": "A", "qty": 5 },
    { "product": "B", "qty": 10 }
  ],
  "mixed": [1, "two", { three: 3 }, [4, 5]]           // Mixed types (valid but avoid)
}

// Query arrays
db.collection.find({ tags: "mongodb" })               // Contains element
db.collection.find({ tags: { $all: ["mongodb", "nosql"] } })  // Contains all
db.collection.find({ scores: { $elemMatch: { $gte: 90 } } })  // Element matches
db.collection.find({ "items.product": "A" })          // Embedded field

// Array size
db.collection.find({ tags: { $size: 3 } })            // Exact size
```

---

## 4. Document Size and Limits

### Size Limits

```
┌─────────────────────────────────────────────────────────────────────┐
│                    MongoDB Document Limits                           │
│                                                                      │
│  Maximum document size:     16 MB (BSON)                            │
│  Maximum nesting depth:     100 levels                              │
│  Maximum field name length: ~1000 bytes                             │
│  Maximum index key size:    1024 bytes                              │
│  Maximum indexes/collection: 64                                      │
│                                                                      │
│  For larger files: Use GridFS                                        │
│  GridFS splits files into 255KB chunks                              │
└─────────────────────────────────────────────────────────────────────┘
```

### Checking Document Size

```javascript
// Check document size
Object.bsonsize({ name: "John", age: 30 })
// Returns size in bytes

// In mongosh
db.collection.find().forEach(function(doc) {
  print(doc._id + ": " + Object.bsonsize(doc) + " bytes")
})

// Find large documents
db.collection.aggregate([
  { $project: {
    size: { $bsonSize: "$$ROOT" },
    doc: "$$ROOT"
  }},
  { $match: { size: { $gt: 1000000 } } },  // > 1MB
  { $sort: { size: -1 } }
])
```

---

## 5. Schema Validation

### JSON Schema Validation

```javascript
// Create collection with validation
db.createCollection("users", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["name", "email", "age"],
      properties: {
        name: {
          bsonType: "string",
          description: "must be a string and is required"
        },
        email: {
          bsonType: "string",
          pattern: "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$",
          description: "must be a valid email"
        },
        age: {
          bsonType: "int",
          minimum: 0,
          maximum: 150,
          description: "must be an integer between 0 and 150"
        },
        status: {
          enum: ["active", "inactive", "pending"],
          description: "must be one of the allowed values"
        },
        address: {
          bsonType: "object",
          required: ["city", "country"],
          properties: {
            street: { bsonType: "string" },
            city: { bsonType: "string" },
            country: { bsonType: "string" }
          }
        }
      }
    }
  },
  validationLevel: "strict",    // strict, moderate
  validationAction: "error"     // error, warn
})

// Validation levels:
// strict: Apply to all inserts and updates
// moderate: Apply only to valid existing documents

// Validation actions:
// error: Reject invalid documents
// warn: Allow but log warning
```

### Modify Existing Validation

```javascript
// Add or update validation
db.runCommand({
  collMod: "users",
  validator: {
    $jsonSchema: {
      // ... new schema
    }
  },
  validationLevel: "moderate"
})

// Remove validation
db.runCommand({
  collMod: "users",
  validator: {},
  validationLevel: "off"
})
```

---

## 6. Data Type Conversions

### Type Coercion in Aggregation

```javascript
// Convert types in aggregation pipeline
db.collection.aggregate([
  {
    $project: {
      // String to integer
      ageInt: { $toInt: "$ageString" },

      // Number to string
      idString: { $toString: "$_id" },

      // String to date
      dateValue: { $toDate: "$dateString" },

      // To boolean
      isActive: { $toBool: "$status" },

      // Safe conversion (returns null on error)
      safeInt: {
        $convert: {
          input: "$value",
          to: "int",
          onError: null,
          onNull: 0
        }
      }
    }
  }
])

// $convert supported types:
// double, string, objectId, bool, date, int, long, decimal
```

### Handling Null and Missing

```javascript
// Check for null
db.collection.find({ field: null })           // field is null OR missing

// Check for existence
db.collection.find({ field: { $exists: true } })   // field exists
db.collection.find({ field: { $exists: false } })  // field missing

// Check for null specifically
db.collection.find({
  field: { $type: "null" }                    // field is explicitly null
})

// $ifNull in aggregation
db.collection.aggregate([
  {
    $project: {
      value: { $ifNull: ["$optionalField", "default"] }
    }
  }
])
```

---

## 7. Best Practices

### Document Design

```javascript
// Good: Flat structure for simple queries
{
  "_id": ObjectId("..."),
  "firstName": "John",
  "lastName": "Doe",
  "email": "john@example.com"
}

// Good: Embedded when data is accessed together
{
  "_id": ObjectId("..."),
  "name": "John Doe",
  "address": {
    "street": "123 Main St",
    "city": "NYC"
  }
}

// Avoid: Unbounded arrays
{
  "userId": 1,
  "posts": [/* could grow to millions */]
}

// Better: Reference for large/unbounded data
// Users collection
{ "_id": 1, "name": "John" }

// Posts collection
{ "_id": 101, "userId": 1, "content": "..." }
```

### Field Naming

```javascript
// Good: Clear, consistent naming
{
  "firstName": "John",
  "lastName": "Doe",
  "createdAt": ISODate("..."),
  "updatedAt": ISODate("...")
}

// Consider: Short names for storage efficiency
// (when document count is huge)
{
  "fn": "John",    // firstName
  "ln": "Doe",     // lastName
  "ca": ISODate("...")  // createdAt
}

// Avoid: Reserved characters
{
  "user.name": "x",     // Bad: dot in name
  "$price": 100         // Bad: starts with $
}
```

---

## Summary

| Type | BSON Type # | Description |
|------|-------------|-------------|
| ObjectId | 7 | 12-byte unique identifier |
| String | 2 | UTF-8 text |
| Int32 | 16 | 32-bit integer |
| Int64 | 18 | 64-bit integer |
| Double | 1 | 64-bit float |
| Decimal128 | 19 | 128-bit decimal |
| Date | 9 | UTC datetime |
| Boolean | 8 | true/false |
| Null | 10 | null value |
| Array | 4 | Ordered list |
| Object | 3 | Embedded document |

---

## Further Reading

- MongoDB BSON Types documentation
- JSON Schema Validation
- Document Size and WiredTiger
