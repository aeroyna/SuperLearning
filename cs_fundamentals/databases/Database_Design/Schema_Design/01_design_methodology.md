# Design Methodology

## Requirements Analysis

```
┌─────────────────────────────────────────────────────────────────┐
│              Gathering Requirements                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  FUNCTIONAL REQUIREMENTS                                        │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Questions to ask:                                          │ │
│  │ • What data needs to be stored?                            │ │
│  │ • What are the main entities?                              │ │
│  │ • How are entities related?                                │ │
│  │ • What operations will be performed?                       │ │
│  │ • What queries will be most common?                        │ │
│  │ • What reports are needed?                                 │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  NON-FUNCTIONAL REQUIREMENTS                                    │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Expected data volume (rows, size)                        │ │
│  │ • Growth rate (daily/monthly)                              │ │
│  │ • Performance requirements (latency, throughput)           │ │
│  │ • Availability requirements                                │ │
│  │ • Consistency requirements                                 │ │
│  │ • Retention policies                                       │ │
│  │ • Compliance requirements (GDPR, HIPAA)                    │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ACCESS PATTERN ANALYSIS                                        │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Document for each major operation:                         │ │
│  │                                                             │ │
│  │ Operation: Get user profile                                │ │
│  │ Frequency: 10,000/second                                   │ │
│  │ Latency requirement: < 10ms                                │ │
│  │ Data accessed: users, addresses, preferences               │ │
│  │                                                             │ │
│  │ Operation: Search products                                 │ │
│  │ Frequency: 5,000/second                                    │ │
│  │ Latency requirement: < 100ms                               │ │
│  │ Filters: category, price, brand, attributes                │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Entity-Relationship Modeling

```
┌─────────────────────────────────────────────────────────────────┐
│              ER Diagram Notation                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ENTITIES                                                       │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ ┌─────────────┐                                            │ │
│  │ │   Entity    │  Rectangle = Entity (table)               │ │
│  │ └─────────────┘                                            │ │
│  │                                                             │ │
│  │ ╔═════════════╗                                            │ │
│  │ ║ Weak Entity ║  Double border = Weak entity              │ │
│  │ ╚═════════════╝  (depends on another entity)              │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  RELATIONSHIPS                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ One-to-One (1:1)                                           │ │
│  │ [User]────────────[Profile]                                │ │
│  │                                                             │ │
│  │ One-to-Many (1:N)                                          │ │
│  │ [Customer]────────<[Orders]                                │ │
│  │                                                             │ │
│  │ Many-to-Many (M:N)                                         │ │
│  │ [Students]>──────<[Courses]                                │ │
│  │        (requires junction table)                           │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  CARDINALITY NOTATION                                           │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ ──||──  Exactly one                                        │ │
│  │ ──|<──  One or many                                        │ │
│  │ ──o|──  Zero or one                                        │ │
│  │ ──o<──  Zero or many                                       │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  EXAMPLE: E-Commerce ER Diagram                                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  ┌──────────┐     ┌───────────┐     ┌──────────────┐       │ │
│  │  │ Customer │──|<─│   Order   │──|<─│ Order_Item   │       │ │
│  │  └──────────┘     └───────────┘     └──────────────┘       │ │
│  │       │                                    │                │ │
│  │       │                                    │                │ │
│  │       ▼                                    ▼                │ │
│  │  ┌──────────┐                       ┌──────────────┐       │ │
│  │  │ Address  │                       │   Product    │       │ │
│  │  └──────────┘                       └──────────────┘       │ │
│  │                                            │                │ │
│  │                                            ▼                │ │
│  │                                     ┌──────────────┐       │ │
│  │                                     │  Category    │       │ │
│  │                                     └──────────────┘       │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## From ER to Tables

```
┌─────────────────────────────────────────────────────────────────┐
│              Converting ER to Relational Schema                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ENTITY → TABLE                                                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Entity: Customer                                           │ │
│  │ Attributes: id, name, email, phone                         │ │
│  │                                                             │ │
│  │ CREATE TABLE customers (                                   │ │
│  │     id BIGINT PRIMARY KEY,                                 │ │
│  │     name VARCHAR(100) NOT NULL,                            │ │
│  │     email VARCHAR(255) UNIQUE NOT NULL,                    │ │
│  │     phone VARCHAR(20)                                      │ │
│  │ );                                                          │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ONE-TO-MANY → FOREIGN KEY                                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Customer (1) ──── (N) Orders                               │ │
│  │                                                             │ │
│  │ CREATE TABLE orders (                                      │ │
│  │     id BIGINT PRIMARY KEY,                                 │ │
│  │     customer_id BIGINT REFERENCES customers(id),           │ │
│  │     order_date TIMESTAMP,                                  │ │
│  │     total DECIMAL(12,2)                                    │ │
│  │ );                                                          │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  MANY-TO-MANY → JUNCTION TABLE                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Students (M) ──── (N) Courses                              │ │
│  │                                                             │ │
│  │ CREATE TABLE enrollments (                                 │ │
│  │     student_id BIGINT REFERENCES students(id),             │ │
│  │     course_id BIGINT REFERENCES courses(id),               │ │
│  │     enrolled_at TIMESTAMP DEFAULT NOW(),                   │ │
│  │     grade VARCHAR(2),                                      │ │
│  │     PRIMARY KEY (student_id, course_id)                    │ │
│  │ );                                                          │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ONE-TO-ONE → SAME TABLE OR SEPARATE                            │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Option A: Single table (if always accessed together)      │ │
│  │ CREATE TABLE users (                                       │ │
│  │     id BIGINT PRIMARY KEY,                                 │ │
│  │     email VARCHAR(255),                                    │ │
│  │     -- profile fields inline                               │ │
│  │     bio TEXT,                                              │ │
│  │     avatar_url VARCHAR(500)                                │ │
│  │ );                                                          │ │
│  │                                                             │ │
│  │ Option B: Separate tables (if often accessed separately)  │ │
│  │ CREATE TABLE user_profiles (                               │ │
│  │     user_id BIGINT PRIMARY KEY REFERENCES users(id),       │ │
│  │     bio TEXT,                                              │ │
│  │     avatar_url VARCHAR(500)                                │ │
│  │ );                                                          │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Normalization Process

```
┌─────────────────────────────────────────────────────────────────┐
│              Applying Normal Forms                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  UNNORMALIZED DATA                                              │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ order_id │ customer │ email        │ items                 │ │
│  │ ─────────┼──────────┼──────────────┼────────────────────── │ │
│  │ 1        │ Alice    │ a@mail.com   │ Book:$10, Pen:$2     │ │
│  │ 2        │ Alice    │ a@mail.com   │ Notebook:$5          │ │
│  │                                                             │ │
│  │ Problems: Repeating groups, redundancy                     │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  1NF: ATOMIC VALUES, NO REPEATING GROUPS                        │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ order_id │ customer │ email       │ item     │ price      │ │
│  │ ─────────┼──────────┼─────────────┼──────────┼─────────── │ │
│  │ 1        │ Alice    │ a@mail.com  │ Book     │ $10        │ │
│  │ 1        │ Alice    │ a@mail.com  │ Pen      │ $2         │ │
│  │ 2        │ Alice    │ a@mail.com  │ Notebook │ $5         │ │
│  │                                                             │ │
│  │ ✓ Each cell has single value                              │ │
│  │ ✗ Still has redundancy                                    │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  2NF: NO PARTIAL DEPENDENCIES                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Orders:                                                    │ │
│  │ order_id │ customer_id │ order_date                        │ │
│  │                                                             │ │
│  │ Order_Items:                                               │ │
│  │ order_id │ item_id │ quantity │ price                     │ │
│  │                                                             │ │
│  │ ✓ Non-key attributes depend on whole key                  │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  3NF: NO TRANSITIVE DEPENDENCIES                                │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Customers:                                                  │ │
│  │ customer_id │ name │ email                                 │ │
│  │                                                             │ │
│  │ Orders:                                                    │ │
│  │ order_id │ customer_id │ order_date                        │ │
│  │                                                             │ │
│  │ ✓ No non-key → non-key dependencies                       │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Index Design

```
┌─────────────────────────────────────────────────────────────────┐
│              Index Strategy                                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  INDEX SELECTION GUIDELINES                                     │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Always index:                                               │ │
│  │ • Primary keys (automatic)                                 │ │
│  │ • Foreign keys (for joins)                                 │ │
│  │ • Columns in WHERE clauses                                 │ │
│  │ • Columns in ORDER BY                                      │ │
│  │ • Columns in JOIN conditions                               │ │
│  │                                                             │ │
│  │ Consider indexing:                                          │ │
│  │ • High-cardinality columns                                 │ │
│  │ • Frequently filtered columns                              │ │
│  │ • Covering indexes for common queries                      │ │
│  │                                                             │ │
│  │ Avoid indexing:                                             │ │
│  │ • Low-cardinality columns (boolean, status)                │ │
│  │ • Frequently updated columns                               │ │
│  │ • Wide columns (long text)                                 │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  COMPOSITE INDEX DESIGN                                         │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ -- Common query:                                           │ │
│  │ SELECT * FROM orders                                       │ │
│  │ WHERE customer_id = ? AND status = ?                       │ │
│  │ ORDER BY created_at DESC;                                  │ │
│  │                                                             │ │
│  │ -- Optimal composite index:                                │ │
│  │ CREATE INDEX idx_orders_customer_status_created            │ │
│  │ ON orders(customer_id, status, created_at DESC);           │ │
│  │                                                             │ │
│  │ Column order matters:                                      │ │
│  │ 1. Equality columns first (customer_id, status)           │ │
│  │ 2. Range/sort columns last (created_at)                   │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```
