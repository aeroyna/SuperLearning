# Analytics and Data Warehousing

## OLTP vs OLAP

```
┌─────────────────────────────────────────────────────────────────┐
│              OLTP vs OLAP Comparison                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│                    OLTP                    OLAP                 │
│  ┌──────────────────────────┐  ┌──────────────────────────┐    │
│  │ Purpose:                 │  │ Purpose:                 │    │
│  │ Operational transactions │  │ Analytical queries       │    │
│  │                          │  │                          │    │
│  │ Queries:                 │  │ Queries:                 │    │
│  │ Simple, affecting few    │  │ Complex, scanning        │    │
│  │ rows                     │  │ millions of rows         │    │
│  │                          │  │                          │    │
│  │ Access Pattern:          │  │ Access Pattern:          │    │
│  │ Random read/write        │  │ Sequential scans         │    │
│  │                          │  │                          │    │
│  │ Data:                    │  │ Data:                    │    │
│  │ Current state            │  │ Historical data          │    │
│  │                          │  │                          │    │
│  │ Schema:                  │  │ Schema:                  │    │
│  │ Normalized (3NF)         │  │ Denormalized (Star/Snow) │    │
│  │                          │  │                          │    │
│  │ Example DB:              │  │ Example DB:              │    │
│  │ PostgreSQL, MySQL        │  │ Snowflake, BigQuery,     │    │
│  │                          │  │ ClickHouse, Redshift     │    │
│  └──────────────────────────┘  └──────────────────────────┘    │
│                                                                  │
│  Example Query Comparison:                                      │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ OLTP: Get order details                                  │  │
│  │ SELECT * FROM orders WHERE id = 12345;                   │  │
│  │ → Returns 1 row, uses primary key index                  │  │
│  │                                                           │  │
│  │ OLAP: Monthly sales by region                            │  │
│  │ SELECT region, SUM(amount)                               │  │
│  │ FROM orders                                               │  │
│  │ WHERE order_date BETWEEN '2024-01-01' AND '2024-12-31'  │  │
│  │ GROUP BY region;                                          │  │
│  │ → Scans millions of rows, aggregates                     │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

## Data Warehouse Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│              Modern Data Warehouse Architecture                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                    Data Sources                          │    │
│  │  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐           │    │
│  │  │  OLTP  │ │  APIs  │ │  Logs  │ │  Files │           │    │
│  │  │   DBs  │ │        │ │        │ │  (S3)  │           │    │
│  │  └────────┘ └────────┘ └────────┘ └────────┘           │    │
│  └────────────────────────┬────────────────────────────────┘    │
│                           │                                      │
│                           ▼                                      │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                   Ingestion Layer                        │    │
│  │  ┌──────────────────────────────────────────────────┐   │    │
│  │  │ Batch: Airflow, dbt, Fivetran                    │   │    │
│  │  │ Stream: Kafka, Kinesis, Debezium                 │   │    │
│  │  └──────────────────────────────────────────────────┘   │    │
│  └────────────────────────┬────────────────────────────────┘    │
│                           │                                      │
│                           ▼                                      │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                   Storage Layer                          │    │
│  │  ┌──────────────────────────────────────────────────┐   │    │
│  │  │ Data Lake: S3, GCS, ADLS (Parquet, ORC)         │   │    │
│  │  │ Data Warehouse: Snowflake, BigQuery, Redshift   │   │    │
│  │  └──────────────────────────────────────────────────┘   │    │
│  └────────────────────────┬────────────────────────────────┘    │
│                           │                                      │
│                           ▼                                      │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                 Transformation Layer                     │    │
│  │  ┌──────────────────────────────────────────────────┐   │    │
│  │  │ dbt, Spark, SQL transformations                  │   │    │
│  │  │ Bronze → Silver → Gold (Medallion Architecture)  │   │    │
│  │  └──────────────────────────────────────────────────┘   │    │
│  └────────────────────────┬────────────────────────────────┘    │
│                           │                                      │
│                           ▼                                      │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                  Consumption Layer                       │    │
│  │  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐           │    │
│  │  │   BI   │ │ Adhoc  │ │   ML   │ │  Apps  │           │    │
│  │  │ Tools  │ │ Query  │ │Training│ │        │           │    │
│  │  └────────┘ └────────┘ └────────┘ └────────┘           │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

## Dimensional Modeling

```
┌─────────────────────────────────────────────────────────────────┐
│              Star Schema                                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│                    ┌──────────────────┐                         │
│                    │   dim_product    │                         │
│                    │──────────────────│                         │
│                    │ product_id (PK)  │                         │
│                    │ name             │                         │
│                    │ category         │                         │
│                    │ brand            │                         │
│                    └────────┬─────────┘                         │
│                             │                                    │
│  ┌──────────────────┐       │       ┌──────────────────┐        │
│  │   dim_customer   │       │       │    dim_date      │        │
│  │──────────────────│       │       │──────────────────│        │
│  │ customer_id (PK) │       │       │ date_id (PK)     │        │
│  │ name             │       │       │ date             │        │
│  │ segment          │       │       │ month            │        │
│  │ region           │       │       │ quarter          │        │
│  └────────┬─────────┘       │       │ year             │        │
│           │                 │       └────────┬─────────┘        │
│           │                 │                │                   │
│           │    ┌────────────┴────────────┐   │                  │
│           │    │       fact_sales        │   │                  │
│           │    │─────────────────────────│   │                  │
│           └───>│ customer_id (FK)        │<──┘                  │
│                │ product_id (FK)         │                      │
│                │ date_id (FK)            │                      │
│                │ store_id (FK)           │                      │
│                │ quantity                │                      │
│                │ unit_price              │                      │
│                │ total_amount            │                      │
│                └─────────────────────────┘                      │
│                             │                                    │
│                    ┌────────┴─────────┐                         │
│                    │    dim_store     │                         │
│                    │──────────────────│                         │
│                    │ store_id (PK)    │                         │
│                    │ name             │                         │
│                    │ city             │                         │
│                    │ state            │                         │
│                    └──────────────────┘                         │
│                                                                  │
│  Benefits:                                                      │
│  • Simple queries (few joins)                                   │
│  • Optimized for aggregations                                   │
│  • Easy to understand                                           │
└─────────────────────────────────────────────────────────────────┘
```

## Columnar Storage

```
┌─────────────────────────────────────────────────────────────────┐
│              Row vs Column Storage                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ROW STORAGE (PostgreSQL, MySQL)                                │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Row 1: [id=1, name="Alice", age=30, city="NYC"]            │ │
│  │ Row 2: [id=2, name="Bob", age=25, city="LA"]               │ │
│  │ Row 3: [id=3, name="Carol", age=35, city="NYC"]            │ │
│  │                                                             │ │
│  │ ✓ Fast for single row access                               │ │
│  │ ✓ Efficient for OLTP                                       │ │
│  │ ✗ Slow for column aggregations                             │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  COLUMN STORAGE (ClickHouse, Parquet, Redshift)                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ id column:   [1, 2, 3, ...]                                │ │
│  │ name column: ["Alice", "Bob", "Carol", ...]                │ │
│  │ age column:  [30, 25, 35, ...]                             │ │
│  │ city column: ["NYC", "LA", "NYC", ...]                     │ │
│  │                                                             │ │
│  │ ✓ Only reads needed columns                                │ │
│  │ ✓ Better compression (similar values together)             │ │
│  │ ✓ SIMD vectorized operations                               │ │
│  │ ✗ Slow for single row access                               │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  Query: SELECT AVG(age) FROM users WHERE city = 'NYC';         │
│                                                                  │
│  Row storage:  Read all columns for matching rows              │
│  Column storage: Read only 'age' and 'city' columns            │
│                  10x-100x faster for analytics                  │
└─────────────────────────────────────────────────────────────────┘
```

## Modern Data Warehouse Solutions

```
┌─────────────────────────────────────────────────────────────────┐
│              Data Warehouse Comparison                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  SNOWFLAKE                                                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Cloud-native, multi-cloud                                │ │
│  │ • Separation of compute and storage                        │ │
│  │ • Automatic scaling                                        │ │
│  │ • Time travel (query historical data)                      │ │
│  │ • Data sharing across accounts                             │ │
│  │ • Pay per query                                            │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  GOOGLE BIGQUERY                                                │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Serverless, no infrastructure management                 │ │
│  │ • Columnar storage + Dremel query engine                   │ │
│  │ • Built-in ML (BigQuery ML)                                │ │
│  │ • Streaming inserts                                        │ │
│  │ • BI Engine for fast BI queries                            │ │
│  │ • Pay per TB scanned                                       │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  CLICKHOUSE                                                     │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Open source, self-hosted or cloud                        │ │
│  │ • Extremely fast for aggregations                          │ │
│  │ • Real-time analytics                                      │ │
│  │ • MergeTree engine family                                  │ │
│  │ • Good for time-series analytics                           │ │
│  │ • Lower cost at scale                                      │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  APACHE DRUID                                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Real-time analytics database                             │ │
│  │ • Sub-second OLAP queries                                  │ │
│  │ • Time-series optimized                                    │ │
│  │ • High concurrency                                         │ │
│  │ • Stream ingestion (Kafka native)                          │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## ETL vs ELT

```
┌─────────────────────────────────────────────────────────────────┐
│              ETL vs ELT Patterns                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ETL (Extract, Transform, Load) - Traditional                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Source → [Transform Engine] → Data Warehouse               │ │
│  │           (Informatica, Talend)                            │ │
│  │                                                             │ │
│  │ Transform before loading                                    │ │
│  │ ✓ Clean data in warehouse                                  │ │
│  │ ✗ Transformation bottleneck                                │ │
│  │ ✗ Lose raw data                                            │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ELT (Extract, Load, Transform) - Modern                       │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Source → Data Lake/Warehouse → [Transform in place]        │ │
│  │           (Fivetran, Airbyte)      (dbt, Spark)            │ │
│  │                                                             │ │
│  │ Load raw, transform later                                   │ │
│  │ ✓ Keep raw data                                            │ │
│  │ ✓ Leverage warehouse compute                               │ │
│  │ ✓ More flexible                                            │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  MEDALLION ARCHITECTURE (Databricks pattern)                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  ┌─────────┐    ┌─────────┐    ┌─────────┐               │ │
│  │  │ Bronze  │ → │ Silver  │ → │  Gold   │               │ │
│  │  │  (Raw)  │    │(Cleaned)│    │(Business│               │ │
│  │  │         │    │         │    │ Ready) │               │ │
│  │  └─────────┘    └─────────┘    └─────────┘               │ │
│  │                                                             │ │
│  │  Bronze: Raw data as-is from sources                       │ │
│  │  Silver: Cleaned, deduplicated, typed                      │ │
│  │  Gold: Business-level aggregates, facts/dims              │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Best Practices

```
┌─────────────────────────────────────────────────────────────────┐
│              Data Warehouse Best Practices                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  PARTITIONING                                                   │
│  • Partition by date (most common)                              │
│  • Enables partition pruning                                    │
│  • Makes data lifecycle management easier                       │
│                                                                  │
│  CLUSTERING/SORTING                                             │
│  • Sort by commonly filtered columns                            │
│  • Improves compression                                         │
│  • Speeds up range queries                                      │
│                                                                  │
│  MATERIALIZED VIEWS                                             │
│  • Pre-compute expensive aggregations                           │
│  • Trade storage for query speed                                │
│  • Refresh on schedule or incrementally                         │
│                                                                  │
│  DATA QUALITY                                                   │
│  • Implement data contracts                                     │
│  • Add data quality tests (Great Expectations, dbt tests)       │
│  • Monitor for anomalies                                        │
│                                                                  │
│  COST OPTIMIZATION                                              │
│  • Use appropriate data types                                   │
│  • Compress data (Parquet, ORC)                                 │
│  • Archive old data to cold storage                             │
│  • Monitor query costs                                          │
└─────────────────────────────────────────────────────────────────┘
```
