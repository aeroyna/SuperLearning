# Temporal Data Modeling

## Introduction to Temporal Data

```
┌─────────────────────────────────────────────────────────────────┐
│              Types of Temporal Data                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  TRANSACTION TIME (System Time)                                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ When the data was stored in the database                   │ │
│  │ • Managed by the system                                    │ │
│  │ • Used for auditing, versioning                            │ │
│  │ • "What did we know at time T?"                            │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  VALID TIME (Application Time)                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ When the data was true in the real world                   │ │
│  │ • Managed by application                                   │ │
│  │ • Used for scheduling, history                             │ │
│  │ • "What was true at time T?"                               │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  BI-TEMPORAL                                                    │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Tracks both transaction and valid time                     │ │
│  │ • Full audit capability                                    │ │
│  │ • "What did we think was true at time T1 about time T2?"  │ │
│  │ • Most complex but most powerful                           │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  EXAMPLE: Employee Salary                                       │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Jan 1: Salary set to $50,000 (valid from Jan 1)           │ │
│  │ Mar 15: Correction - salary was actually $52,000          │ │
│  │                                                             │ │
│  │ Transaction time: When we recorded each fact              │ │
│  │ Valid time: When salary was actually in effect            │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Slowly Changing Dimensions (SCD)

```
┌─────────────────────────────────────────────────────────────────┐
│              SCD Types                                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  TYPE 1: OVERWRITE                                              │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Simply update the record, no history kept                  │ │
│  │                                                             │ │
│  │ Before:  id=1, name="John", city="NYC"                    │ │
│  │ After:   id=1, name="John", city="LA"                     │ │
│  │                                                             │ │
│  │ Use when: History not needed                               │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  TYPE 2: ADD NEW ROW                                            │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Keep full history with version rows                        │ │
│  │                                                             │ │
│  │ CREATE TABLE customers (                                   │ │
│  │     id BIGINT,                -- business key             │ │
│  │     version INT,                                           │ │
│  │     name VARCHAR(100),                                     │ │
│  │     city VARCHAR(100),                                     │ │
│  │     valid_from DATE,                                       │ │
│  │     valid_to DATE,            -- NULL = current           │ │
│  │     is_current BOOLEAN,                                    │ │
│  │     PRIMARY KEY (id, version)                              │ │
│  │ );                                                          │ │
│  │                                                             │ │
│  │ id │ ver │ name │ city │ valid_from │ valid_to │ current  │ │
│  │ ───┼─────┼──────┼──────┼────────────┼──────────┼───────── │ │
│  │ 1  │ 1   │ John │ NYC  │ 2020-01-01 │ 2023-06-30│ false   │ │
│  │ 1  │ 2   │ John │ LA   │ 2023-07-01 │ NULL     │ true    │ │
│  │                                                             │ │
│  │ Use when: Full history needed for reporting                │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  TYPE 3: ADD NEW COLUMN                                         │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Keep limited history (previous value only)                 │ │
│  │                                                             │ │
│  │ id │ name │ current_city │ previous_city │ city_changed   │ │
│  │ ───┼──────┼──────────────┼───────────────┼─────────────── │ │
│  │ 1  │ John │ LA           │ NYC           │ 2023-07-01     │ │
│  │                                                             │ │
│  │ Use when: Only need previous value                         │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  TYPE 4: HISTORY TABLE                                          │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Separate current and history tables                        │ │
│  │                                                             │ │
│  │ customers (current only):                                  │ │
│  │ id │ name │ city                                           │ │
│  │ ───┼──────┼──────                                          │ │
│  │ 1  │ John │ LA                                             │ │
│  │                                                             │ │
│  │ customers_history (all versions):                          │ │
│  │ id │ name │ city │ valid_from │ valid_to                  │ │
│  │ ───┼──────┼──────┼────────────┼───────────                 │ │
│  │ 1  │ John │ NYC  │ 2020-01-01 │ 2023-06-30                │ │
│  │ 1  │ John │ LA   │ 2023-07-01 │ NULL                      │ │
│  │                                                             │ │
│  │ Use when: Current queries must be fast                     │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  TYPE 6: HYBRID (1+2+3)                                         │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Combines multiple approaches                               │ │
│  │ • Keeps full history (Type 2)                             │ │
│  │ • Has current flag and previous value columns             │ │
│  │ • Most flexible but most complex                          │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Temporal Table Implementation

```
┌─────────────────────────────────────────────────────────────────┐
│              SQL:2011 Temporal Tables                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  SYSTEM-VERSIONED TABLES (PostgreSQL approach)                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ -- Main table with system columns                          │ │
│  │ CREATE TABLE employees (                                   │ │
│  │     id BIGINT PRIMARY KEY,                                 │ │
│  │     name VARCHAR(100),                                     │ │
│  │     department VARCHAR(100),                               │ │
│  │     salary DECIMAL(10,2),                                  │ │
│  │     sys_start TIMESTAMP GENERATED ALWAYS AS ROW START,    │ │
│  │     sys_end TIMESTAMP GENERATED ALWAYS AS ROW END,        │ │
│  │     PERIOD FOR SYSTEM_TIME (sys_start, sys_end)           │ │
│  │ ) WITH SYSTEM VERSIONING;                                  │ │
│  │                                                             │ │
│  │ -- Query current data                                      │ │
│  │ SELECT * FROM employees;                                   │ │
│  │                                                             │ │
│  │ -- Query as of specific time                               │ │
│  │ SELECT * FROM employees                                    │ │
│  │ FOR SYSTEM_TIME AS OF '2023-01-01 00:00:00';              │ │
│  │                                                             │ │
│  │ -- Query range                                             │ │
│  │ SELECT * FROM employees                                    │ │
│  │ FOR SYSTEM_TIME BETWEEN '2023-01-01' AND '2023-12-31';    │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  MANUAL IMPLEMENTATION (Works everywhere)                       │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ CREATE TABLE employees (                                   │ │
│  │     id BIGINT,                                             │ │
│  │     name VARCHAR(100),                                     │ │
│  │     department VARCHAR(100),                               │ │
│  │     salary DECIMAL(10,2),                                  │ │
│  │     valid_from TIMESTAMP NOT NULL DEFAULT NOW(),           │ │
│  │     valid_to TIMESTAMP,                                    │ │
│  │     PRIMARY KEY (id, valid_from)                           │ │
│  │ );                                                          │ │
│  │                                                             │ │
│  │ -- Current records view                                    │ │
│  │ CREATE VIEW current_employees AS                           │ │
│  │ SELECT * FROM employees WHERE valid_to IS NULL;            │ │
│  │                                                             │ │
│  │ -- Update procedure (close old, insert new)                │ │
│  │ UPDATE employees SET valid_to = NOW()                      │ │
│  │ WHERE id = 1 AND valid_to IS NULL;                        │ │
│  │                                                             │ │
│  │ INSERT INTO employees (id, name, department, salary)       │ │
│  │ VALUES (1, 'John', 'Engineering', 75000);                  │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Event Sourcing

```
┌─────────────────────────────────────────────────────────────────┐
│              Event Sourcing Pattern                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  CONCEPT                                                        │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Store events instead of current state                      │ │
│  │ Current state = replay of all events                       │ │
│  │                                                             │ │
│  │ Traditional:  Account.balance = 1000                       │ │
│  │                                                             │ │
│  │ Event sourced:                                              │ │
│  │   Event 1: AccountOpened(initial_balance=500)             │ │
│  │   Event 2: MoneyDeposited(amount=700)                     │ │
│  │   Event 3: MoneyWithdrawn(amount=200)                     │ │
│  │   Current balance: 500 + 700 - 200 = 1000                 │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  SCHEMA                                                         │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ CREATE TABLE events (                                      │ │
│  │     id UUID PRIMARY KEY DEFAULT gen_random_uuid(),         │ │
│  │     aggregate_type VARCHAR(100) NOT NULL,                  │ │
│  │     aggregate_id VARCHAR(100) NOT NULL,                    │ │
│  │     event_type VARCHAR(100) NOT NULL,                      │ │
│  │     event_data JSONB NOT NULL,                             │ │
│  │     metadata JSONB,                                        │ │
│  │     version INT NOT NULL,                                  │ │
│  │     created_at TIMESTAMP DEFAULT NOW(),                    │ │
│  │     UNIQUE (aggregate_type, aggregate_id, version)         │ │
│  │ );                                                          │ │
│  │                                                             │ │
│  │ -- Sample events                                           │ │
│  │ aggregate_type │ aggregate_id │ event_type     │ data      │ │
│  │ ───────────────┼──────────────┼────────────────┼────────── │ │
│  │ Account        │ acc-123      │ AccountOpened  │ {bal:500} │ │
│  │ Account        │ acc-123      │ MoneyDeposited │ {amt:700} │ │
│  │ Account        │ acc-123      │ MoneyWithdrawn │ {amt:200} │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  PROJECTIONS (Materialized Views)                               │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ -- Denormalized read model, updated by event handlers     │ │
│  │ CREATE TABLE account_balances (                            │ │
│  │     account_id VARCHAR(100) PRIMARY KEY,                   │ │
│  │     balance DECIMAL(12,2),                                 │ │
│  │     last_event_id UUID,                                    │ │
│  │     updated_at TIMESTAMP                                   │ │
│  │ );                                                          │ │
│  │                                                             │ │
│  │ Benefits:                                                   │ │
│  │ ✓ Complete audit trail                                     │ │
│  │ ✓ Can rebuild state at any point                          │ │
│  │ ✓ Multiple projections from same events                   │ │
│  │                                                             │ │
│  │ Challenges:                                                 │ │
│  │ ✗ Complexity                                               │ │
│  │ ✗ Eventual consistency for projections                    │ │
│  │ ✗ Event schema evolution                                  │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Time-Series Optimization

```
┌─────────────────────────────────────────────────────────────────┐
│              Time-Series Schema Design                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  PARTITIONING BY TIME                                           │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ CREATE TABLE metrics (                                     │ │
│  │     time TIMESTAMPTZ NOT NULL,                             │ │
│  │     device_id TEXT,                                        │ │
│  │     metric_name TEXT,                                      │ │
│  │     value DOUBLE PRECISION                                 │ │
│  │ ) PARTITION BY RANGE (time);                               │ │
│  │                                                             │ │
│  │ -- Monthly partitions                                      │ │
│  │ CREATE TABLE metrics_2024_01 PARTITION OF metrics          │ │
│  │ FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');          │ │
│  │                                                             │ │
│  │ Benefits:                                                   │ │
│  │ • Fast partition pruning for time-range queries           │ │
│  │ • Easy data retention (drop old partitions)               │ │
│  │ • Parallel query across partitions                        │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  TIMESCALEDB HYPERTABLES                                        │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ -- Automatic partitioning                                  │ │
│  │ SELECT create_hypertable('metrics', 'time');               │ │
│  │                                                             │ │
│  │ -- Compression for old data                                │ │
│  │ ALTER TABLE metrics SET (                                  │ │
│  │     timescaledb.compress,                                  │ │
│  │     timescaledb.compress_segmentby = 'device_id'          │ │
│  │ );                                                          │ │
│  │                                                             │ │
│  │ -- Continuous aggregates                                   │ │
│  │ CREATE MATERIALIZED VIEW metrics_hourly                    │ │
│  │ WITH (timescaledb.continuous) AS                           │ │
│  │ SELECT time_bucket('1 hour', time) AS hour,                │ │
│  │        device_id,                                          │ │
│  │        AVG(value) as avg_value                             │ │
│  │ FROM metrics                                                │ │
│  │ GROUP BY hour, device_id;                                  │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```
