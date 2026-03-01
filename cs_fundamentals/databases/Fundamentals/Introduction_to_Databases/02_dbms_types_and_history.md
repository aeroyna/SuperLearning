# DBMS Types and History

## 1. Evolution of Database Systems

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        DATABASE EVOLUTION TIMELINE                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  1960s          1970s          1980s          1990s         2000s+          │
│    │              │              │              │              │             │
│    ▼              ▼              ▼              ▼              ▼             │
│ ┌──────┐     ┌──────────┐   ┌──────────┐   ┌─────────┐   ┌──────────┐       │
│ │File  │     │Relational│   │Object-   │   │Object-  │   │NoSQL     │       │
│ │Systems│────▶│ DBMS    │───▶│Oriented │───▶│Relational│──▶│NewSQL   │       │
│ └──────┘     └──────────┘   │  DBMS    │   │  DBMS   │   │Cloud DB  │       │
│                             └──────────┘   └─────────┘   └──────────┘       │
│    │              │                                                          │
│    ▼              ▼                                                          │
│ Hierarchical  Network                                                        │
│    DBMS        DBMS                                                          │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Historical Database Models

### 2.1 Hierarchical Database Model (1960s)

Data organized in a tree-like structure with parent-child relationships.

```
                    ┌─────────────┐
                    │  Company    │
                    └──────┬──────┘
                           │
          ┌────────────────┼────────────────┐
          │                │                │
    ┌─────▼─────┐    ┌─────▼─────┐    ┌─────▼─────┐
    │Department │    │Department │    │Department │
    │    HR     │    │  Finance  │    │Engineering│
    └─────┬─────┘    └─────┬─────┘    └─────┬─────┘
          │                │                │
    ┌─────▼─────┐    ┌─────▼─────┐    ┌─────▼─────┐
    │ Employee  │    │ Employee  │    │ Employee  │
    │   John    │    │   Jane    │    │    Bob    │
    └───────────┘    └───────────┘    └───────────┘
```

**Example: IBM IMS (Information Management System)**

```
* Characteristics:
  - One-to-many relationships only
  - Fast for hierarchical queries
  - Data redundancy required for many-to-many
  - Navigation through parent-child links

* Limitations:
  - Cannot model many-to-many relationships naturally
  - Complex queries require procedural navigation
  - Schema changes are difficult
```

### 2.2 Network Database Model (1960s-1970s)

Extension of hierarchical model allowing many-to-many relationships.

```
    ┌───────────┐         ┌───────────┐
    │  Student  │         │  Course   │
    │   John    │◄───────►│   Math    │
    └─────┬─────┘         └─────┬─────┘
          │                     │
          │    ┌───────────┐    │
          └───►│Enrollment │◄───┘
               │  Record   │
               └───────────┘
```

**Example: CODASYL (Conference on Data Systems Languages)**

```
* Characteristics:
  - Set-based relationships
  - Multiple parent types allowed
  - More flexible than hierarchical
  - Still requires navigation

* Limitations:
  - Complex pointer management
  - Query language is procedural
  - Difficult to understand and maintain
```

---

## 3. Relational Database Model (1970s - Present)

Proposed by **Edgar F. Codd** at IBM in 1970. Based on relational algebra and set theory.

### Core Principles

```sql
-- Data represented as relations (tables)
-- Each relation is a set of tuples (rows)
-- Operations based on relational algebra

┌──────────────────────────────────────────────────────────────┐
│                    RELATIONAL ALGEBRA                         │
├──────────────────────────────────────────────────────────────┤
│  σ (Selection)    - Filter rows based on condition           │
│  π (Projection)   - Select specific columns                  │
│  ∪ (Union)        - Combine two relations                    │
│  ∩ (Intersection) - Common tuples in both relations          │
│  − (Difference)   - Tuples in first but not in second       │
│  × (Cartesian)    - All combinations of tuples               │
│  ⋈ (Join)         - Combine related tuples                   │
└──────────────────────────────────────────────────────────────┘
```

### Example Schema

```sql
-- Students table
CREATE TABLE students (
    student_id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(255)
);

-- Courses table
CREATE TABLE courses (
    course_id INT PRIMARY KEY,
    title VARCHAR(200),
    credits INT
);

-- Enrollments (junction table for many-to-many)
CREATE TABLE enrollments (
    student_id INT,
    course_id INT,
    grade CHAR(1),
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES students(student_id),
    FOREIGN KEY (course_id) REFERENCES courses(course_id)
);
```

### Codd's 12 Rules

| Rule | Name | Description |
|------|------|-------------|
| 0 | Foundation | Must use relational facilities exclusively |
| 1 | Information | All data in tables with rows and columns |
| 2 | Guaranteed Access | Every value accessible by table+column+key |
| 3 | Null Values | Systematic treatment of missing values |
| 4 | Catalog | Database description in relational form |
| 5 | Comprehensive Language | At least one comprehensive data language |
| 6 | View Updating | Theoretically updatable views are updatable |
| 7 | Insert/Update/Delete | Set-level operations for all modifications |
| 8 | Physical Independence | Apps unaffected by storage changes |
| 9 | Logical Independence | Apps unaffected by logical changes |
| 10 | Integrity | Constraints definable in catalog |
| 11 | Distribution | Distribution transparent to users |
| 12 | Non-Subversion | Cannot bypass constraints via low-level |

---

## 4. Types of Modern DBMS

### 4.1 Relational DBMS (RDBMS)

```
┌─────────────────────────────────────────────────────────────┐
│                    POPULAR RDBMS                             │
├──────────────┬──────────────────────────────────────────────┤
│ Oracle       │ Enterprise, PL/SQL, RAC clustering           │
│ MySQL        │ Open source, InnoDB, widely used             │
│ PostgreSQL   │ Advanced features, extensions, MVCC          │
│ SQL Server   │ Microsoft, T-SQL, tight Windows integration  │
│ SQLite       │ Embedded, serverless, file-based             │
│ MariaDB      │ MySQL fork, Aria storage engine              │
└──────────────┴──────────────────────────────────────────────┘
```

**Use Cases:**
- Structured data with clear relationships
- ACID transaction requirements
- Complex queries and reporting
- Financial systems, ERP, CRM

### 4.2 NoSQL Databases

#### Document Stores

```json
// MongoDB document example
{
    "_id": ObjectId("507f1f77bcf86cd799439011"),
    "name": "John Doe",
    "email": "john@example.com",
    "orders": [
        {
            "order_id": "ORD001",
            "items": ["Widget A", "Widget B"],
            "total": 99.99
        }
    ],
    "preferences": {
        "newsletter": true,
        "theme": "dark"
    }
}
```

**Examples:** MongoDB, CouchDB, Amazon DocumentDB

**Use Cases:**
- Flexible schema requirements
- Content management systems
- Catalogs with varying attributes
- Rapid prototyping

#### Key-Value Stores

```
┌─────────────────────────────────────────────────────────┐
│                   KEY-VALUE STORE                        │
├─────────────────────┬───────────────────────────────────┤
│        KEY          │              VALUE                 │
├─────────────────────┼───────────────────────────────────┤
│ user:1001           │ {"name":"John","email":"j@x.com"} │
│ session:abc123      │ {"user_id":1001,"expires":...}    │
│ cache:homepage      │ "<html>...</html>"                │
│ counter:pageviews   │ 1547832                           │
└─────────────────────┴───────────────────────────────────┘
```

**Examples:** Redis, Amazon DynamoDB, Memcached, etcd

**Use Cases:**
- Session management
- Caching
- Real-time analytics
- Leaderboards
- Rate limiting

#### Wide-Column Stores

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        WIDE-COLUMN STRUCTURE                             │
├─────────────────────────────────────────────────────────────────────────┤
│ Row Key: user:john                                                       │
│ ┌─────────────────┬─────────────────┬─────────────────────────────────┐ │
│ │  Column Family  │  Column Family  │      Column Family              │ │
│ │    "profile"    │    "orders"     │        "activity"               │ │
│ ├─────────────────┼─────────────────┼─────────────────────────────────┤ │
│ │ name: "John"    │ 2023-01: $99    │ 2023-01-15: login               │ │
│ │ email: "j@x.c"  │ 2023-02: $150   │ 2023-01-15: purchase            │ │
│ │ age: 30         │ 2023-03: $75    │ 2023-01-16: logout              │ │
│ └─────────────────┴─────────────────┴─────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────┘
```

**Examples:** Apache Cassandra, HBase, ScyllaDB, Google Bigtable

**Use Cases:**
- Time-series data
- IoT sensor data
- Event logging
- Recommendation engines

#### Graph Databases

```
        ┌─────────┐                    ┌─────────┐
        │  Alice  │──── KNOWS ────────▶│   Bob   │
        │ (User)  │                    │ (User)  │
        └────┬────┘                    └────┬────┘
             │                              │
             │ WORKS_AT                     │ WORKS_AT
             │                              │
             ▼                              ▼
        ┌─────────┐                    ┌─────────┐
        │  Acme   │◀──── PARTNER ─────│  Corp   │
        │(Company)│                    │(Company)│
        └─────────┘                    └─────────┘
```

**Examples:** Neo4j, Amazon Neptune, JanusGraph, ArangoDB

**Use Cases:**
- Social networks
- Fraud detection
- Knowledge graphs
- Recommendation systems
- Network topology

### 4.3 NewSQL Databases

Combine SQL/ACID guarantees with NoSQL scalability.

```
┌─────────────────────────────────────────────────────────────────────┐
│                         NewSQL CHARACTERISTICS                       │
├─────────────────────────────────────────────────────────────────────┤
│  ✓ SQL interface and relational model                               │
│  ✓ ACID transactions across distributed nodes                       │
│  ✓ Horizontal scalability (add nodes to scale)                      │
│  ✓ Automatic sharding and rebalancing                               │
│  ✓ High availability with automatic failover                        │
└─────────────────────────────────────────────────────────────────────┘
```

**Examples:** CockroachDB, TiDB, Google Spanner, YugabyteDB, Vitess

### 4.4 Time-Series Databases

Optimized for time-stamped data.

```sql
-- TimescaleDB (PostgreSQL extension) example
CREATE TABLE sensor_data (
    time        TIMESTAMPTZ NOT NULL,
    sensor_id   INTEGER,
    temperature DOUBLE PRECISION,
    humidity    DOUBLE PRECISION
);

-- Create hypertable for automatic partitioning
SELECT create_hypertable('sensor_data', 'time');

-- Query with time-based aggregation
SELECT time_bucket('1 hour', time) AS hour,
       sensor_id,
       AVG(temperature) as avg_temp
FROM sensor_data
WHERE time > NOW() - INTERVAL '24 hours'
GROUP BY hour, sensor_id;
```

**Examples:** InfluxDB, TimescaleDB, Prometheus, QuestDB

### 4.5 Search Databases

Optimized for full-text search and analytics.

```json
// Elasticsearch document and query
PUT /products/_doc/1
{
    "name": "Wireless Bluetooth Headphones",
    "description": "High-quality audio with noise cancellation",
    "price": 149.99,
    "category": "Electronics"
}

GET /products/_search
{
    "query": {
        "multi_match": {
            "query": "wireless headphones",
            "fields": ["name^2", "description"]
        }
    }
}
```

**Examples:** Elasticsearch, Apache Solr, Meilisearch, Typesense

---

## 5. DBMS Comparison Matrix

| Feature | RDBMS | Document | Key-Value | Wide-Column | Graph |
|---------|-------|----------|-----------|-------------|-------|
| **Schema** | Fixed | Flexible | None | Column families | Nodes/Edges |
| **Scalability** | Vertical | Horizontal | Horizontal | Horizontal | Varies |
| **ACID** | Full | Document-level | Limited | Tunable | Full |
| **Joins** | Native | Limited | None | Limited | Native |
| **Query Language** | SQL | Custom/SQL-like | Simple get/set | CQL | Cypher/Gremlin |
| **Best For** | Structured data | Variable data | Caching | Time-series | Relationships |

---

## 6. Historical Milestones

| Year | Event |
|------|-------|
| 1960 | Charles Bachman creates first DBMS (IDS) |
| 1970 | Edgar Codd publishes relational model paper |
| 1974 | IBM develops System R (first SQL implementation) |
| 1979 | Oracle releases first commercial RDBMS |
| 1983 | IBM releases DB2 |
| 1989 | Microsoft SQL Server 1.0 |
| 1995 | MySQL released as open source |
| 1996 | PostgreSQL 6.0 (renamed from Postgres95) |
| 2007 | Amazon Dynamo paper published |
| 2009 | MongoDB founded |
| 2010 | Redis gains popularity |
| 2012 | Google Spanner paper published |
| 2015 | CockroachDB founded |
| 2017 | Cloud-native databases become mainstream |

---

## 7. Choosing the Right DBMS

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DECISION FLOWCHART                                │
└─────────────────────────────────────────────────────────────────────┘

                        ┌─────────────────┐
                        │ What's your     │
                        │ data structure? │
                        └────────┬────────┘
                                 │
        ┌────────────────────────┼────────────────────────┐
        │                        │                        │
        ▼                        ▼                        ▼
  ┌───────────┐          ┌───────────┐          ┌───────────────┐
  │Structured │          │Semi-struct│          │ Relationships │
  │ Tables    │          │ JSON/XML  │          │   are Key     │
  └─────┬─────┘          └─────┬─────┘          └───────┬───────┘
        │                      │                        │
        ▼                      ▼                        ▼
  ┌───────────┐          ┌───────────┐          ┌───────────────┐
  │Need ACID? │          │ Document  │          │ Graph DB      │
  │  Yes/No   │          │   Store   │          │ (Neo4j)       │
  └─────┬─────┘          │ (MongoDB) │          └───────────────┘
        │                └───────────┘
   ┌────┴────┐
   ▼         ▼
 ┌────┐   ┌─────┐
 │Yes │   │ No  │
 └─┬──┘   └──┬──┘
   │         │
   ▼         ▼
┌──────┐  ┌────────────┐
│RDBMS │  │Need speed? │
│MySQL │  └──────┬─────┘
│Postgres│       │
└──────┘    ┌────┴────┐
            ▼         ▼
         ┌─────┐  ┌────────┐
         │Cache│  │Columnar│
         │Redis│  │Cassandra│
         └─────┘  └────────┘
```

---

## 8. Summary

| Model | Era | Strength | Weakness |
|-------|-----|----------|----------|
| Hierarchical | 1960s | Fast hierarchical access | Rigid structure |
| Network | 1970s | Many-to-many relationships | Complex navigation |
| Relational | 1970s+ | Flexibility, SQL, ACID | Scaling challenges |
| Document | 2000s+ | Schema flexibility | Weaker consistency |
| Key-Value | 2000s+ | Speed, simplicity | Limited queries |
| Wide-Column | 2000s+ | Write performance | Complex data model |
| Graph | 2000s+ | Relationship queries | General queries slow |
| NewSQL | 2010s+ | SQL + Scale | Newer, less mature |

Understanding database history helps in choosing the right tool for your specific requirements.
