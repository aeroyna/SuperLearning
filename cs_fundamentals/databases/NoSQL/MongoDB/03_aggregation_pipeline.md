# Aggregation Pipeline

## Learning Objectives
- Understand aggregation pipeline concepts
- Master common pipeline stages
- Implement data transformations
- Optimize aggregation performance

---

## 1. Pipeline Fundamentals

### What is the Aggregation Pipeline?

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Aggregation Pipeline                              │
│                                                                      │
│  Documents flow through stages, each stage transforms the data       │
│                                                                      │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐          │
│  │ $match  │───▶│ $group  │───▶│ $sort   │───▶│ $limit  │          │
│  │ Filter  │    │ Aggregate│   │ Order   │    │ Top N   │          │
│  └─────────┘    └─────────┘    └─────────┘    └─────────┘          │
│       │              │              │              │                 │
│       ▼              ▼              ▼              ▼                 │
│  [100 docs]     [10 groups]    [10 sorted]    [5 results]          │
│                                                                      │
│  Key Concepts:                                                       │
│  • Stages process documents one at a time                           │
│  • Output of one stage is input to next                             │
│  • Early $match reduces documents for later stages                  │
│  • Can use indexes in early stages                                  │
└─────────────────────────────────────────────────────────────────────┘
```

### Basic Syntax

```javascript
db.collection.aggregate([
  { $stage1: { /* expression */ } },
  { $stage2: { /* expression */ } },
  { $stage3: { /* expression */ } }
])

// Example: Sales by category
db.orders.aggregate([
  { $match: { status: "completed" } },
  { $group: {
    _id: "$category",
    totalSales: { $sum: "$amount" },
    count: { $sum: 1 }
  }},
  { $sort: { totalSales: -1 } },
  { $limit: 10 }
])
```

---

## 2. Common Stages

### $match - Filter Documents

```javascript
// Filter early to reduce processing
db.orders.aggregate([
  { $match: {
    status: "completed",
    date: { $gte: ISODate("2024-01-01") },
    amount: { $gt: 100 }
  }}
])

// $match uses indexes when at pipeline start
// Same operators as find()
```

### $project - Shape Documents

```javascript
db.users.aggregate([
  { $project: {
    // Include fields
    name: 1,
    email: 1,

    // Exclude _id
    _id: 0,

    // Rename field
    userEmail: "$email",

    // Computed field
    fullName: { $concat: ["$firstName", " ", "$lastName"] },

    // Nested field
    city: "$address.city",

    // Conditional
    status: {
      $cond: {
        if: { $gte: ["$age", 18] },
        then: "adult",
        else: "minor"
      }
    }
  }}
])
```

### $group - Aggregate Values

```javascript
db.orders.aggregate([
  { $group: {
    _id: "$customerId",  // Group key (null for all docs)

    // Accumulators
    totalAmount: { $sum: "$amount" },
    avgAmount: { $avg: "$amount" },
    maxAmount: { $max: "$amount" },
    minAmount: { $min: "$amount" },
    count: { $sum: 1 },

    // Collect values
    orderIds: { $push: "$_id" },
    uniqueProducts: { $addToSet: "$product" },

    // First/last in group
    firstOrder: { $first: "$date" },
    lastOrder: { $last: "$date" }
  }}
])

// Multiple group keys
db.sales.aggregate([
  { $group: {
    _id: {
      year: { $year: "$date" },
      month: { $month: "$date" },
      product: "$product"
    },
    total: { $sum: "$amount" }
  }}
])

// Group all documents
db.orders.aggregate([
  { $group: {
    _id: null,
    totalRevenue: { $sum: "$amount" },
    orderCount: { $sum: 1 }
  }}
])
```

### $sort and $limit

```javascript
// Sort documents
db.products.aggregate([
  { $sort: { price: -1, name: 1 } },  // price DESC, name ASC
  { $limit: 10 }
])

// Top N per group
db.scores.aggregate([
  { $sort: { score: -1 } },
  { $group: {
    _id: "$subject",
    topScores: { $push: "$$ROOT" }
  }},
  { $project: {
    topScores: { $slice: ["$topScores", 3] }  // Top 3 per subject
  }}
])
```

### $unwind - Deconstruct Arrays

```javascript
// Original document:
// { _id: 1, tags: ["a", "b", "c"] }

db.posts.aggregate([
  { $unwind: "$tags" }
])

// Result: 3 documents
// { _id: 1, tags: "a" }
// { _id: 1, tags: "b" }
// { _id: 1, tags: "c" }

// With options
{ $unwind: {
  path: "$tags",
  includeArrayIndex: "tagIndex",      // Add index field
  preserveNullAndEmptyArrays: true    // Keep docs without array
}}

// Common pattern: Count tags
db.posts.aggregate([
  { $unwind: "$tags" },
  { $group: {
    _id: "$tags",
    count: { $sum: 1 }
  }},
  { $sort: { count: -1 } }
])
```

### $lookup - Left Outer Join

```javascript
// Join collections
db.orders.aggregate([
  { $lookup: {
    from: "customers",         // Join with customers collection
    localField: "customerId",  // Field in orders
    foreignField: "_id",       // Field in customers
    as: "customerInfo"         // Output array field
  }},
  { $unwind: "$customerInfo" } // Convert array to object
])

// Pipeline lookup (more powerful)
db.orders.aggregate([
  { $lookup: {
    from: "products",
    let: { productIds: "$items.productId" },
    pipeline: [
      { $match: {
        $expr: { $in: ["$_id", "$$productIds"] }
      }},
      { $project: { name: 1, price: 1 } }
    ],
    as: "products"
  }}
])
```

### $addFields - Add New Fields

```javascript
db.orders.aggregate([
  { $addFields: {
    totalWithTax: { $multiply: ["$total", 1.08] },
    status: "processed",
    processedAt: "$$NOW"  // Current timestamp
  }}
])

// $set is alias for $addFields
db.orders.aggregate([
  { $set: { processed: true } }
])
```

---

## 3. Expression Operators

### Arithmetic

```javascript
{ $add: [expr1, expr2] }           // Addition
{ $subtract: [expr1, expr2] }       // Subtraction
{ $multiply: [expr1, expr2] }       // Multiplication
{ $divide: [expr1, expr2] }         // Division
{ $mod: [expr1, expr2] }            // Modulo
{ $abs: expr }                      // Absolute value
{ $ceil: expr }                     // Ceiling
{ $floor: expr }                    // Floor
{ $round: [expr, places] }          // Round
{ $pow: [base, exponent] }          // Power
{ $sqrt: expr }                     // Square root
```

### String

```javascript
{ $concat: [str1, str2, ...] }      // Concatenate
{ $substr: [str, start, len] }      // Substring
{ $toLower: expr }                  // Lowercase
{ $toUpper: expr }                  // Uppercase
{ $trim: { input: expr } }          // Trim whitespace
{ $split: [str, delimiter] }        // Split to array
{ $strLenCP: expr }                 // String length
{ $regexMatch: {                    // Regex match
  input: str,
  regex: /pattern/,
  options: "i"
}}
```

### Date

```javascript
{ $year: dateExpr }
{ $month: dateExpr }
{ $dayOfMonth: dateExpr }
{ $hour: dateExpr }
{ $minute: dateExpr }
{ $second: dateExpr }
{ $dayOfWeek: dateExpr }            // 1=Sunday, 7=Saturday
{ $dayOfYear: dateExpr }
{ $week: dateExpr }

{ $dateFromString: {
  dateString: "2024-01-15",
  format: "%Y-%m-%d"
}}

{ $dateToString: {
  date: "$createdAt",
  format: "%Y-%m-%d %H:%M"
}}

{ $dateDiff: {
  startDate: "$start",
  endDate: "$end",
  unit: "day"
}}

{ $dateAdd: {
  startDate: "$date",
  unit: "month",
  amount: 1
}}
```

### Conditional

```javascript
// $cond (ternary)
{ $cond: {
  if: boolExpr,
  then: trueExpr,
  else: falseExpr
}}
// Short form: { $cond: [if, then, else] }

// $switch (case statement)
{ $switch: {
  branches: [
    { case: { $eq: ["$status", "A"] }, then: "Active" },
    { case: { $eq: ["$status", "I"] }, then: "Inactive" }
  ],
  default: "Unknown"
}}

// $ifNull
{ $ifNull: [expr, replacement] }  // Use replacement if null

// Null coalescing chain
{ $ifNull: ["$field1", "$field2", "default"] }
```

### Array

```javascript
{ $arrayElemAt: [array, idx] }      // Element at index
{ $first: array }                   // First element
{ $last: array }                    // Last element
{ $size: array }                    // Array length
{ $slice: [array, n] }              // First n elements
{ $slice: [array, skip, n] }        // Skip and take
{ $reverseArray: array }            // Reverse
{ $concatArrays: [arr1, arr2] }     // Concatenate
{ $in: [elem, array] }              // Element in array
{ $isArray: expr }                  // Is array?

// $map - Transform each element
{ $map: {
  input: "$items",
  as: "item",
  in: { $multiply: ["$$item.qty", "$$item.price"] }
}}

// $filter - Filter array elements
{ $filter: {
  input: "$items",
  as: "item",
  cond: { $gte: ["$$item.qty", 5] }
}}

// $reduce - Reduce array to single value
{ $reduce: {
  input: "$items",
  initialValue: 0,
  in: { $add: ["$$value", "$$this.qty"] }
}}
```

---

## 4. Advanced Stages

### $bucket - Histogram

```javascript
db.products.aggregate([
  { $bucket: {
    groupBy: "$price",
    boundaries: [0, 25, 50, 100, 500],
    default: "Other",
    output: {
      count: { $sum: 1 },
      products: { $push: "$name" }
    }
  }}
])

// Auto bucket
db.products.aggregate([
  { $bucketAuto: {
    groupBy: "$price",
    buckets: 5,
    output: {
      count: { $sum: 1 },
      avgPrice: { $avg: "$price" }
    }
  }}
])
```

### $facet - Multiple Pipelines

```javascript
// Run multiple pipelines in parallel
db.products.aggregate([
  { $facet: {
    // Price statistics
    priceStats: [
      { $group: {
        _id: null,
        avgPrice: { $avg: "$price" },
        minPrice: { $min: "$price" },
        maxPrice: { $max: "$price" }
      }}
    ],
    // Top categories
    topCategories: [
      { $group: { _id: "$category", count: { $sum: 1 } } },
      { $sort: { count: -1 } },
      { $limit: 5 }
    ],
    // Recent products
    recent: [
      { $sort: { createdAt: -1 } },
      { $limit: 5 },
      { $project: { name: 1, price: 1 } }
    ]
  }}
])
```

### $graphLookup - Recursive Lookup

```javascript
// Traverse hierarchical data
db.employees.aggregate([
  { $match: { name: "CEO" } },
  { $graphLookup: {
    from: "employees",
    startWith: "$_id",
    connectFromField: "_id",
    connectToField: "managerId",
    as: "allReports",
    maxDepth: 5,
    depthField: "level"
  }}
])
```

### $merge and $out

```javascript
// Write results to collection
db.orders.aggregate([
  { $group: {
    _id: "$customerId",
    totalSpent: { $sum: "$amount" }
  }},
  { $merge: {
    into: "customer_stats",
    on: "_id",
    whenMatched: "replace",
    whenNotMatched: "insert"
  }}
])

// Replace entire collection
db.orders.aggregate([
  { $group: { _id: "$product", count: { $sum: 1 } } },
  { $out: "product_counts" }
])
```

### $setWindowFields - Window Functions

```javascript
// Running totals, rankings, etc.
db.sales.aggregate([
  { $setWindowFields: {
    partitionBy: "$product",
    sortBy: { date: 1 },
    output: {
      runningTotal: {
        $sum: "$amount",
        window: { documents: ["unbounded", "current"] }
      },
      rank: {
        $rank: {}
      },
      movingAvg: {
        $avg: "$amount",
        window: { documents: [-2, 0] }  // Last 3 docs
      }
    }
  }}
])
```

---

## 5. Optimization

### Use Indexes

```javascript
// $match at start can use indexes
db.orders.aggregate([
  { $match: { status: "completed", date: { $gte: ISODate("2024-01-01") } } },
  // ... rest of pipeline
])

// Check if index is used
db.orders.aggregate([...]).explain("executionStats")
```

### Reduce Early

```javascript
// Good: Filter and project early
db.orders.aggregate([
  { $match: { status: "completed" } },      // Filter first
  { $project: { customerId: 1, amount: 1 } }, // Only needed fields
  { $group: { _id: "$customerId", total: { $sum: "$amount" } } }
])

// Bad: Project and group all fields
db.orders.aggregate([
  { $group: { _id: "$customerId", orders: { $push: "$$ROOT" } } },
  { $match: { "orders.status": "completed" } }  // Too late!
])
```

### Allow Disk Use

```javascript
// For large datasets exceeding 100MB memory limit
db.largeCollection.aggregate([
  { $group: { _id: "$field", count: { $sum: 1 } } }
], { allowDiskUse: true })
```

---

## 6. Practical Examples

### Sales Report

```javascript
db.orders.aggregate([
  { $match: {
    date: { $gte: ISODate("2024-01-01"), $lt: ISODate("2025-01-01") }
  }},
  { $group: {
    _id: {
      year: { $year: "$date" },
      month: { $month: "$date" }
    },
    revenue: { $sum: "$total" },
    orders: { $sum: 1 },
    avgOrder: { $avg: "$total" }
  }},
  { $sort: { "_id.year": 1, "_id.month": 1 } },
  { $project: {
    _id: 0,
    month: { $concat: [
      { $toString: "$_id.year" }, "-",
      { $toString: "$_id.month" }
    ]},
    revenue: { $round: ["$revenue", 2] },
    orders: 1,
    avgOrder: { $round: ["$avgOrder", 2] }
  }}
])
```

### Customer Segmentation

```javascript
db.customers.aggregate([
  { $lookup: {
    from: "orders",
    localField: "_id",
    foreignField: "customerId",
    as: "orders"
  }},
  { $addFields: {
    totalSpent: { $sum: "$orders.total" },
    orderCount: { $size: "$orders" }
  }},
  { $addFields: {
    segment: {
      $switch: {
        branches: [
          { case: { $gte: ["$totalSpent", 10000] }, then: "VIP" },
          { case: { $gte: ["$totalSpent", 1000] }, then: "Regular" }
        ],
        default: "Occasional"
      }
    }
  }},
  { $group: {
    _id: "$segment",
    customers: { $sum: 1 },
    avgSpent: { $avg: "$totalSpent" }
  }}
])
```

---

## Summary

| Stage | Purpose |
|-------|---------|
| $match | Filter documents |
| $project | Shape documents |
| $group | Aggregate values |
| $sort | Order documents |
| $limit/$skip | Pagination |
| $unwind | Deconstruct arrays |
| $lookup | Join collections |
| $facet | Multiple pipelines |
| $bucket | Histogram grouping |
| $merge/$out | Write results |

---

## Further Reading

- MongoDB Aggregation Pipeline documentation
- Aggregation Pipeline Optimization
- Quick Reference for Operators
