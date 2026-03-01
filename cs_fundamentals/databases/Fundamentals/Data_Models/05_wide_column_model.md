# Wide-Column Model

## 1. Introduction

The **wide-column model** (also called column-family or extensible record stores) organizes data into rows and column families, where each row can have a different set of columns. It's optimized for write-heavy workloads and horizontal scalability.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      WIDE-COLUMN MODEL STRUCTURE                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Row Key: user:alice                                                        │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │ Column Family: profile     │ Column Family: activity                │   │
│   ├────────────┬───────────────┼────────────────────┬───────────────────┤   │
│   │ name:Alice │ email:a@x.com │ 2024-01-15:login   │ 2024-01-15:view   │   │
│   │ age:28     │ city:NYC      │ 2024-01-14:logout  │ 2024-01-14:click  │   │
│   └────────────┴───────────────┴────────────────────┴───────────────────┘   │
│                                                                              │
│   Row Key: user:bob                                                          │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │ Column Family: profile     │ Column Family: activity                │   │
│   ├────────────┬───────────────┼────────────────────┬───────────────────┤   │
│   │ name:Bob   │ email:b@x.com │ 2024-01-15:login   │                   │   │
│   │            │ phone:555-123 │                    │                   │   │
│   └────────────┴───────────────┴────────────────────┴───────────────────┘   │
│                                                                              │
│   KEY CONCEPTS:                                                              │
│   • Partition Key: Determines data distribution across nodes                │
│   • Clustering Key: Determines sort order within partition                  │
│   • Column Family: Logical grouping of columns                              │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Core Concepts

### 2.1 Data Organization

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        CASSANDRA DATA HIERARCHY                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   KEYSPACE (Database)                                                        │
│   └── TABLE (Column Family)                                                  │
│       └── PARTITION (Row group by partition key)                            │
│           └── ROW (Unique by clustering key)                                │
│               └── COLUMN (Name-value-timestamp)                             │
│                                                                              │
│   Example:                                                                   │
│   ┌─────────────────────────────────────────────────────────────────┐       │
│   │ Keyspace: ecommerce                                              │       │
│   │ ├── Table: orders                                                │       │
│   │ │   ├── Partition: customer_id = 'C001'                         │       │
│   │ │   │   ├── Row: order_id = 'O001' (columns: date, total, ...)  │       │
│   │ │   │   └── Row: order_id = 'O002' (columns: date, total, ...)  │       │
│   │ │   └── Partition: customer_id = 'C002'                         │       │
│   │ │       └── Row: order_id = 'O003' (columns: date, total, ...)  │       │
│   └─────────────────────────────────────────────────────────────────┘       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Primary Key Components

```cql
-- Cassandra CQL
CREATE TABLE orders (
    customer_id UUID,          -- Partition key
    order_date DATE,           -- Clustering key 1
    order_id UUID,             -- Clustering key 2
    total DECIMAL,
    status TEXT,
    items LIST<FROZEN<item_type>>,
    PRIMARY KEY ((customer_id), order_date, order_id)
) WITH CLUSTERING ORDER BY (order_date DESC, order_id ASC);

-- Partition Key: Determines which node stores the data
--   - All rows with same partition key are stored together
--   - Enables fast lookups by partition key
--   - Data distribution across cluster

-- Clustering Key: Determines sort order within partition
--   - Enables efficient range queries
--   - Supports ORDER BY on clustering columns

-- Compound Partition Key (multiple columns)
CREATE TABLE sensor_data (
    sensor_id UUID,
    date DATE,
    hour INT,
    minute INT,
    reading DOUBLE,
    PRIMARY KEY ((sensor_id, date), hour, minute)
);
```

### 2.3 Column Families and Tables

```cql
-- Create keyspace with replication
CREATE KEYSPACE ecommerce WITH replication = {
    'class': 'NetworkTopologyStrategy',
    'dc1': 3,
    'dc2': 2
};

USE ecommerce;

-- Static columns (one value per partition)
CREATE TABLE users (
    user_id UUID,
    email TEXT STATIC,           -- Same for all rows in partition
    username TEXT STATIC,
    device_id UUID,
    last_login TIMESTAMP,
    PRIMARY KEY (user_id, device_id)
);

-- Collections
CREATE TABLE products (
    product_id UUID PRIMARY KEY,
    name TEXT,
    tags SET<TEXT>,              -- Unordered unique values
    prices MAP<TEXT, DECIMAL>,   -- Key-value pairs
    reviews LIST<TEXT>           -- Ordered, allows duplicates
);

-- User-Defined Types (UDT)
CREATE TYPE address (
    street TEXT,
    city TEXT,
    zip TEXT,
    country TEXT
);

CREATE TABLE customers (
    customer_id UUID PRIMARY KEY,
    name TEXT,
    shipping_address FROZEN<address>,
    billing_address FROZEN<address>
);
```

---

## 3. Data Modeling

### 3.1 Query-Driven Design

Wide-column databases require modeling based on access patterns, not entities:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    QUERY-DRIVEN MODELING PROCESS                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   1. Identify Access Patterns (Queries)                                     │
│      ├── Q1: Get user's orders by date range                               │
│      ├── Q2: Get order details by order ID                                 │
│      └── Q3: Get all orders for a product                                  │
│                                                                              │
│   2. Create Table Per Query Pattern                                         │
│      ├── orders_by_user (for Q1)                                           │
│      ├── orders_by_id (for Q2)                                             │
│      └── orders_by_product (for Q3)                                        │
│                                                                              │
│   3. Denormalize Data (duplicate across tables)                            │
│      └── Same order data in multiple tables                                │
│                                                                              │
│   RULE: One table per query pattern                                         │
│   TRADE-OFF: Storage space vs query performance                            │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 3.2 Denormalization Patterns

```cql
-- Query 1: Get orders by customer (sorted by date)
CREATE TABLE orders_by_customer (
    customer_id UUID,
    order_date TIMESTAMP,
    order_id UUID,
    total DECIMAL,
    status TEXT,
    PRIMARY KEY ((customer_id), order_date, order_id)
) WITH CLUSTERING ORDER BY (order_date DESC);

-- Query 2: Get order by ID (full details)
CREATE TABLE orders_by_id (
    order_id UUID PRIMARY KEY,
    customer_id UUID,
    customer_name TEXT,          -- Denormalized from customers table
    order_date TIMESTAMP,
    total DECIMAL,
    status TEXT,
    items LIST<FROZEN<order_item>>
);

-- Query 3: Get orders containing a product
CREATE TABLE orders_by_product (
    product_id UUID,
    order_date TIMESTAMP,
    order_id UUID,
    customer_id UUID,
    quantity INT,
    PRIMARY KEY ((product_id), order_date, order_id)
) WITH CLUSTERING ORDER BY (order_date DESC);
```

### 3.3 Time-Series Pattern

```cql
-- Sensor readings with time-based partitioning
CREATE TABLE sensor_readings (
    sensor_id UUID,
    date DATE,                   -- Partition by day
    reading_time TIMESTAMP,
    temperature DOUBLE,
    humidity DOUBLE,
    pressure DOUBLE,
    PRIMARY KEY ((sensor_id, date), reading_time)
) WITH CLUSTERING ORDER BY (reading_time DESC);

-- Query: Get readings for a sensor on a specific day
SELECT * FROM sensor_readings
WHERE sensor_id = ? AND date = '2024-01-15'
AND reading_time >= '2024-01-15 10:00:00'
AND reading_time < '2024-01-15 11:00:00';

-- Bucketing pattern for high-frequency data
CREATE TABLE metrics_bucketed (
    metric_name TEXT,
    bucket_id INT,               -- Partition by time bucket
    timestamp TIMESTAMP,
    value DOUBLE,
    PRIMARY KEY ((metric_name, bucket_id), timestamp)
) WITH CLUSTERING ORDER BY (timestamp DESC);

-- Bucket ID calculation: timestamp_ms / bucket_size_ms
-- Example: hourly buckets = timestamp / 3600000
```

### 3.4 Materialized Views

```cql
-- Base table
CREATE TABLE orders (
    order_id UUID,
    customer_id UUID,
    product_id UUID,
    order_date TIMESTAMP,
    total DECIMAL,
    PRIMARY KEY (order_id)
);

-- Materialized view for different access pattern
CREATE MATERIALIZED VIEW orders_by_customer AS
    SELECT * FROM orders
    WHERE customer_id IS NOT NULL
    AND order_id IS NOT NULL
    PRIMARY KEY (customer_id, order_date, order_id)
    WITH CLUSTERING ORDER BY (order_date DESC);

-- Note: MVs have limitations - prefer manual denormalization for
-- complex cases or when write performance is critical
```

---

## 4. CQL Operations

### 4.1 CRUD Operations

```cql
-- INSERT (or upsert)
INSERT INTO users (user_id, email, name, created_at)
VALUES (uuid(), 'alice@example.com', 'Alice', toTimestamp(now()));

-- INSERT with TTL (auto-delete after time)
INSERT INTO sessions (session_id, user_id, data)
VALUES (uuid(), ?, ?)
USING TTL 3600;  -- Expires in 1 hour

-- INSERT IF NOT EXISTS (lightweight transaction)
INSERT INTO users (user_id, email, name)
VALUES (uuid(), 'alice@example.com', 'Alice')
IF NOT EXISTS;

-- UPDATE
UPDATE users SET name = 'Alice Johnson', updated_at = toTimestamp(now())
WHERE user_id = ?;

-- UPDATE with conditions (lightweight transaction)
UPDATE users SET status = 'inactive'
WHERE user_id = ?
IF status = 'active';

-- DELETE
DELETE FROM users WHERE user_id = ?;

-- DELETE specific columns
DELETE email, phone FROM users WHERE user_id = ?;

-- DELETE with TTL (column-level)
UPDATE users USING TTL 86400
SET temp_token = 'abc123'
WHERE user_id = ?;
```

### 4.2 Querying

```cql
-- Basic select
SELECT * FROM users WHERE user_id = ?;

-- Select specific columns
SELECT name, email FROM users WHERE user_id = ?;

-- Range queries on clustering columns
SELECT * FROM orders_by_customer
WHERE customer_id = ?
AND order_date >= '2024-01-01'
AND order_date < '2024-02-01';

-- LIMIT and ordering
SELECT * FROM orders_by_customer
WHERE customer_id = ?
ORDER BY order_date DESC
LIMIT 10;

-- IN clause (use sparingly)
SELECT * FROM orders_by_id
WHERE order_id IN (uuid1, uuid2, uuid3);

-- ALLOW FILTERING (use with caution - full scan)
SELECT * FROM users
WHERE name = 'Alice'
ALLOW FILTERING;  -- Avoid in production!

-- Token-based pagination
SELECT * FROM users
WHERE token(user_id) > token(?)
LIMIT 100;
```

### 4.3 Batch Operations

```cql
-- Logged batch (atomic within partition)
BEGIN BATCH
    INSERT INTO orders_by_customer (customer_id, order_date, order_id, total)
    VALUES (?, ?, ?, ?);

    INSERT INTO orders_by_id (order_id, customer_id, order_date, total)
    VALUES (?, ?, ?, ?);

    INSERT INTO orders_by_product (product_id, order_date, order_id, quantity)
    VALUES (?, ?, ?, ?);
APPLY BATCH;

-- Unlogged batch (better performance, no atomicity guarantee)
BEGIN UNLOGGED BATCH
    -- Multiple inserts to same partition
    INSERT INTO sensor_readings (sensor_id, date, reading_time, value) VALUES (...);
    INSERT INTO sensor_readings (sensor_id, date, reading_time, value) VALUES (...);
    INSERT INTO sensor_readings (sensor_id, date, reading_time, value) VALUES (...);
APPLY BATCH;

-- Counter batch
BEGIN COUNTER BATCH
    UPDATE page_views SET count = count + 1 WHERE page_id = ?;
    UPDATE user_stats SET total_views = total_views + 1 WHERE user_id = ?;
APPLY BATCH;
```

### 4.4 Secondary Indexes

```cql
-- Create secondary index
CREATE INDEX ON users (email);

-- Query using secondary index
SELECT * FROM users WHERE email = 'alice@example.com';

-- SASI index (more powerful, Cassandra 3.4+)
CREATE CUSTOM INDEX ON users (name)
USING 'org.apache.cassandra.index.sasi.SASIIndex'
WITH OPTIONS = {
    'mode': 'CONTAINS',
    'analyzer_class': 'org.apache.cassandra.index.sasi.analyzer.StandardAnalyzer',
    'case_sensitive': 'false'
};

SELECT * FROM users WHERE name LIKE '%alice%';

-- Note: Secondary indexes have limitations
-- Prefer denormalized tables for frequent queries
```

---

## 5. Consistency and Replication

### 5.1 Consistency Levels

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        CONSISTENCY LEVELS                                    │
├──────────────────┬──────────────────────────────────────────────────────────┤
│ Level            │ Description                                              │
├──────────────────┼──────────────────────────────────────────────────────────┤
│ ANY              │ Write to any node (including hinted handoff)            │
│ ONE              │ Write/read from one replica                              │
│ TWO              │ Write/read from two replicas                             │
│ THREE            │ Write/read from three replicas                           │
│ QUORUM           │ Majority of replicas ((RF/2) + 1)                       │
│ LOCAL_QUORUM     │ Quorum in local datacenter                              │
│ EACH_QUORUM      │ Quorum in each datacenter (writes only)                 │
│ ALL              │ All replicas must respond                               │
│ LOCAL_ONE        │ One replica in local datacenter                         │
│ SERIAL           │ For lightweight transactions (linearizable)             │
│ LOCAL_SERIAL     │ Serial in local datacenter                              │
└──────────────────┴──────────────────────────────────────────────────────────┘

Strong Consistency Formula:
  R + W > N
  Where: R = read consistency, W = write consistency, N = replication factor

Example (RF=3):
  - QUORUM reads (2) + QUORUM writes (2) = 4 > 3 ✓ Strong consistency
  - ONE read (1) + ONE write (1) = 2 < 3 ✗ Eventual consistency
```

### 5.2 Setting Consistency in CQL

```cql
-- Per-query consistency
CONSISTENCY QUORUM;
SELECT * FROM users WHERE user_id = ?;

CONSISTENCY LOCAL_ONE;
INSERT INTO logs (log_id, message) VALUES (?, ?);

-- In application code (example with Python driver)
from cassandra import ConsistencyLevel
from cassandra.query import SimpleStatement

query = SimpleStatement(
    "SELECT * FROM users WHERE user_id = ?",
    consistency_level=ConsistencyLevel.QUORUM
)
```

---

## 6. Code Examples Across Languages

### Python (cassandra-driver)

```python
from cassandra.cluster import Cluster
from cassandra.auth import PlainTextAuthProvider
from cassandra.query import SimpleStatement, BatchStatement
from cassandra import ConsistencyLevel
import uuid

# Connect
auth = PlainTextAuthProvider(username='cassandra', password='cassandra')
cluster = Cluster(['127.0.0.1'], auth_provider=auth)
session = cluster.connect('ecommerce')

# Prepared statements (best practice)
insert_order = session.prepare("""
    INSERT INTO orders_by_customer (customer_id, order_date, order_id, total, status)
    VALUES (?, ?, ?, ?, ?)
""")

select_orders = session.prepare("""
    SELECT * FROM orders_by_customer
    WHERE customer_id = ?
    AND order_date >= ?
    AND order_date < ?
""")

# Insert with prepared statement
customer_id = uuid.UUID('550e8400-e29b-41d4-a716-446655440000')
order_id = uuid.uuid4()
session.execute(insert_order, (customer_id, datetime.now(), order_id, 99.99, 'pending'))

# Query with pagination
from cassandra.query import SimpleStatement

query = SimpleStatement(
    "SELECT * FROM orders_by_customer WHERE customer_id = ?",
    fetch_size=100,
    consistency_level=ConsistencyLevel.QUORUM
)

result = session.execute(query, [customer_id])
for row in result:
    print(f"Order {row.order_id}: {row.total}")

# Batch operations
batch = BatchStatement()
batch.add(insert_order, (customer_id, datetime.now(), uuid.uuid4(), 50.00, 'pending'))
batch.add(insert_order, (customer_id, datetime.now(), uuid.uuid4(), 75.00, 'pending'))
session.execute(batch)

# Async operations
from cassandra.concurrent import execute_concurrent_with_args

queries = [(customer_id, datetime.now(), uuid.uuid4(), i * 10.0, 'pending')
           for i in range(100)]
results = execute_concurrent_with_args(session, insert_order, queries)

cluster.shutdown()
```

### Java (DataStax Driver)

```java
import com.datastax.oss.driver.api.core.CqlSession;
import com.datastax.oss.driver.api.core.cql.*;
import java.net.InetSocketAddress;
import java.time.Instant;
import java.util.UUID;

public class CassandraExample {
    public static void main(String[] args) {
        try (CqlSession session = CqlSession.builder()
                .addContactPoint(new InetSocketAddress("127.0.0.1", 9042))
                .withKeyspace("ecommerce")
                .build()) {

            // Prepared statements
            PreparedStatement insertOrder = session.prepare(
                "INSERT INTO orders_by_customer " +
                "(customer_id, order_date, order_id, total, status) " +
                "VALUES (?, ?, ?, ?, ?)"
            );

            PreparedStatement selectOrders = session.prepare(
                "SELECT * FROM orders_by_customer " +
                "WHERE customer_id = ? AND order_date >= ? AND order_date < ?"
            );

            // Insert
            UUID customerId = UUID.fromString("550e8400-e29b-41d4-a716-446655440000");
            UUID orderId = UUID.randomUUID();

            session.execute(insertOrder.bind(
                customerId,
                Instant.now(),
                orderId,
                new BigDecimal("99.99"),
                "pending"
            ));

            // Query with pagination
            ResultSet rs = session.execute(
                selectOrders.bind(customerId, Instant.parse("2024-01-01T00:00:00Z"),
                                  Instant.parse("2024-02-01T00:00:00Z"))
            );

            for (Row row : rs) {
                System.out.printf("Order %s: %s%n",
                    row.getUuid("order_id"),
                    row.getBigDecimal("total"));
            }

            // Batch
            BatchStatement batch = BatchStatement.newInstance(BatchType.LOGGED)
                .add(insertOrder.bind(customerId, Instant.now(), UUID.randomUUID(),
                     new BigDecimal("50.00"), "pending"))
                .add(insertOrder.bind(customerId, Instant.now(), UUID.randomUUID(),
                     new BigDecimal("75.00"), "pending"));

            session.execute(batch);

            // Async
            CompletionStage<AsyncResultSet> futureResult = session.executeAsync(
                selectOrders.bind(customerId, Instant.now().minusSeconds(86400), Instant.now())
            );
            futureResult.thenAccept(result -> {
                result.currentPage().forEach(row ->
                    System.out.println(row.getUuid("order_id")));
            });
        }
    }
}
```

### JavaScript (cassandra-driver)

```javascript
const cassandra = require('cassandra-driver');
const Uuid = cassandra.types.Uuid;
const TimeUuid = cassandra.types.TimeUuid;

const client = new cassandra.Client({
    contactPoints: ['127.0.0.1'],
    localDataCenter: 'datacenter1',
    keyspace: 'ecommerce'
});

async function main() {
    await client.connect();

    // Prepared statements
    const insertOrder = await client.prepare(`
        INSERT INTO orders_by_customer
        (customer_id, order_date, order_id, total, status)
        VALUES (?, ?, ?, ?, ?)
    `);

    const selectOrders = await client.prepare(`
        SELECT * FROM orders_by_customer
        WHERE customer_id = ?
        AND order_date >= ?
        AND order_date < ?
    `);

    // Insert
    const customerId = Uuid.fromString('550e8400-e29b-41d4-a716-446655440000');
    const orderId = Uuid.random();

    await client.execute(insertOrder, [
        customerId,
        new Date(),
        orderId,
        99.99,
        'pending'
    ], { prepare: true });

    // Query with pagination
    const options = { prepare: true, fetchSize: 100 };
    const result = await client.execute(selectOrders, [
        customerId,
        new Date('2024-01-01'),
        new Date('2024-02-01')
    ], options);

    for (const row of result.rows) {
        console.log(`Order ${row.order_id}: ${row.total}`);
    }

    // Stream large results
    client.stream(selectOrders.query, [customerId, new Date('2024-01-01'), new Date()])
        .on('readable', function () {
            let row;
            while (row = this.read()) {
                console.log(row.order_id);
            }
        })
        .on('end', () => console.log('Done'));

    // Batch
    const queries = [
        { query: insertOrder, params: [customerId, new Date(), Uuid.random(), 50.00, 'pending'] },
        { query: insertOrder, params: [customerId, new Date(), Uuid.random(), 75.00, 'pending'] }
    ];
    await client.batch(queries, { prepare: true });

    await client.shutdown();
}

main().catch(console.error);
```

---

## 7. Advantages and Limitations

### Advantages

| Advantage | Description |
|-----------|-------------|
| **Write Performance** | Optimized for high-volume writes |
| **Linear Scalability** | Add nodes to increase capacity |
| **High Availability** | No single point of failure |
| **Tunable Consistency** | Balance between consistency and performance |
| **Time-Series Friendly** | Excellent for temporal data patterns |
| **Geo-Distribution** | Multi-datacenter support |

### Limitations

| Limitation | Description |
|------------|-------------|
| **Query Flexibility** | Must design tables for specific queries |
| **No Joins** | Data must be denormalized |
| **Learning Curve** | Requires different modeling mindset |
| **Secondary Indexes** | Limited compared to RDBMS |
| **Storage Overhead** | Denormalization increases storage |
| **Ad-hoc Queries** | Not suited for exploratory queries |

---

## 8. When to Use Wide-Column Stores

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        USE CASE DECISION MATRIX                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ✓ USE WIDE-COLUMN                      ✗ DON'T USE WIDE-COLUMN            │
│   ─────────────────                      ──────────────────────              │
│   • Time-series data                     • Complex joins needed              │
│   • Event logging                        • Ad-hoc queries                    │
│   • IoT sensor data                      • Strong transactions               │
│   • User activity tracking               • Small datasets                    │
│   • Write-heavy workloads                • Frequently changing queries       │
│   • Known query patterns                 • Relational data                   │
│   • High availability required           • Need for ACID guarantees         │
│   • Horizontal scaling needed                                                │
│                                                                              │
│   EXAMPLES:                                                                  │
│   • Netflix: Viewing history, recommendations                               │
│   • Apple: iCloud, iMessage                                                 │
│   • Discord: Messages, presence                                             │
│   • Uber: Driver locations, trip data                                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 9. Summary

Wide-column stores like Cassandra and ScyllaDB provide:

- **Horizontal scalability** with linear performance gains
- **High availability** through replication and no single point of failure
- **Excellent write performance** via append-only storage
- **Tunable consistency** from eventual to strong
- **Time-series optimization** for temporal data patterns

They require:
- **Query-first modeling** - design tables around access patterns
- **Denormalization** - duplicate data across tables
- **Careful key design** - partition keys determine distribution
- **Understanding trade-offs** - CAP theorem implications
