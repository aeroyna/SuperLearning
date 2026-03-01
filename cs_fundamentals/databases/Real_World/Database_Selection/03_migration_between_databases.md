# Migration Between Databases

## Migration Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                  Database Migration Types                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  HOMOGENEOUS MIGRATION                                          │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Same database type: MySQL → MySQL, PostgreSQL → PostgreSQL │ │
│  │ • Version upgrades                                         │ │
│  │ • Cloud migration (on-prem → RDS)                          │ │
│  │ • Hardware refresh                                         │ │
│  │ Complexity: Low to Medium                                  │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  HETEROGENEOUS MIGRATION                                        │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Different database types: MySQL → PostgreSQL               │ │
│  │ • Schema translation                                       │ │
│  │ • Data type mapping                                        │ │
│  │ • Query/procedure rewriting                                │ │
│  │ Complexity: High                                           │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  PARADIGM MIGRATION                                             │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Different models: SQL → NoSQL, Monolith → Microservices   │ │
│  │ • Data model redesign                                      │ │
│  │ • Application changes                                      │ │
│  │ • Often gradual/phased                                     │ │
│  │ Complexity: Very High                                      │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Migration Strategies

```
┌─────────────────────────────────────────────────────────────────┐
│                  Migration Approaches                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  BIG BANG MIGRATION                                             │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  [Old DB] ───── Downtime Window ─────► [New DB]            │ │
│  │             export → transform → load                       │ │
│  │                                                             │ │
│  │  ✓ Simple conceptually                                     │ │
│  │  ✓ Clean cut-over                                          │ │
│  │  ✗ Requires downtime                                       │ │
│  │  ✗ High risk (all or nothing)                              │ │
│  │  ✗ Limited rollback options                                │ │
│  │                                                             │ │
│  │  Best for: Small databases, acceptable downtime            │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  PARALLEL RUN (Dual Write)                                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  Application ────┬────► [Old DB] (primary)                 │ │
│  │                  │                                          │ │
│  │                  └────► [New DB] (shadow)                  │ │
│  │                                                             │ │
│  │  Phase 1: Write to both, read from old                     │ │
│  │  Phase 2: Write to both, read from new                     │ │
│  │  Phase 3: Write only to new, decommission old              │ │
│  │                                                             │ │
│  │  ✓ Zero downtime                                           │ │
│  │  ✓ Easy rollback                                           │ │
│  │  ✓ Gradual validation                                      │ │
│  │  ✗ Application complexity                                  │ │
│  │  ✗ Data consistency challenges                             │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  CDC-BASED MIGRATION                                            │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  [Old DB] ──► CDC ──► Kafka ──► [New DB]                   │ │
│  │      │                              │                       │ │
│  │      │     Initial bulk load        │                       │ │
│  │      └──────────────────────────────┘                       │ │
│  │                                                             │ │
│  │  1. Bulk load historical data                              │ │
│  │  2. Start CDC to capture changes                           │ │
│  │  3. Replay changes until caught up                         │ │
│  │  4. Switch traffic                                         │ │
│  │                                                             │ │
│  │  ✓ Minimal downtime (seconds)                              │ │
│  │  ✓ Reliable data transfer                                  │ │
│  │  ✓ Built-in verification                                   │ │
│  │  ✗ Requires CDC infrastructure                             │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  STRANGLER FIG PATTERN                                          │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  ┌─────────────────────────────────────────────────────┐   │ │
│  │  │                    Router/Proxy                      │   │ │
│  │  └───────────────────────┬─────────────────────────────┘   │ │
│  │                          │                                  │ │
│  │            ┌─────────────┼─────────────┐                   │ │
│  │            │             │             │                    │ │
│  │         Feature A    Feature B    Feature C                │ │
│  │            │             │             │                    │ │
│  │         [New DB]     [Old DB]      [Old DB]                │ │
│  │                                                             │ │
│  │  Migrate one feature/table at a time                       │ │
│  │  Old system shrinks as new grows                           │ │
│  │                                                             │ │
│  │  ✓ Low risk per step                                       │ │
│  │  ✓ Learn as you go                                         │ │
│  │  ✓ Can pause/resume                                        │ │
│  │  ✗ Long overall timeline                                   │ │
│  │  ✗ Temporary complexity                                    │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Migration Process

```
┌─────────────────────────────────────────────────────────────────┐
│                  Migration Phases                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  PHASE 1: ASSESSMENT                                            │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Inventory all database objects                           │ │
│  │ • Identify data types needing conversion                   │ │
│  │ • Map stored procedures, triggers, functions               │ │
│  │ • Document dependencies and integrations                   │ │
│  │ • Estimate data volume and growth                          │ │
│  │ • Define success criteria                                  │ │
│  └────────────────────────────────────────────────────────────┘ │
│                          ↓                                       │
│  PHASE 2: SCHEMA CONVERSION                                     │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Convert table definitions                                │ │
│  │ • Map data types                                           │ │
│  │ • Translate constraints and indexes                        │ │
│  │ • Convert stored procedures                                │ │
│  │ • Handle database-specific features                        │ │
│  └────────────────────────────────────────────────────────────┘ │
│                          ↓                                       │
│  PHASE 3: DATA MIGRATION                                        │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Extract data from source                                 │ │
│  │ • Transform data (type conversions, etc.)                  │ │
│  │ • Load into target                                         │ │
│  │ • Set up ongoing sync (if applicable)                      │ │
│  │ • Validate row counts and checksums                        │ │
│  └────────────────────────────────────────────────────────────┘ │
│                          ↓                                       │
│  PHASE 4: APPLICATION MIGRATION                                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Update connection strings                                │ │
│  │ • Rewrite incompatible queries                             │ │
│  │ • Update ORM configurations                                │ │
│  │ • Test all application paths                               │ │
│  │ • Performance testing                                      │ │
│  └────────────────────────────────────────────────────────────┘ │
│                          ↓                                       │
│  PHASE 5: CUTOVER                                               │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Final data sync                                          │ │
│  │ • Switch application to new database                       │ │
│  │ • Monitor for issues                                       │ │
│  │ • Keep rollback ready                                      │ │
│  └────────────────────────────────────────────────────────────┘ │
│                          ↓                                       │
│  PHASE 6: VALIDATION & CLEANUP                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Verify data integrity                                    │ │
│  │ • Confirm application functionality                        │ │
│  │ • Decommission old database (after bake time)              │ │
│  │ • Document lessons learned                                 │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Common Data Type Mappings

```
┌─────────────────────────────────────────────────────────────────┐
│              MySQL to PostgreSQL Type Mapping                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  MySQL Type              PostgreSQL Type                        │
│  ─────────────────────────────────────────────────────────────  │
│  TINYINT                 SMALLINT                               │
│  MEDIUMINT               INTEGER                                │
│  INT                     INTEGER                                │
│  BIGINT                  BIGINT                                 │
│  FLOAT                   REAL                                   │
│  DOUBLE                  DOUBLE PRECISION                       │
│  DECIMAL                 NUMERIC                                │
│  DATETIME                TIMESTAMP                              │
│  TIMESTAMP               TIMESTAMP WITH TIME ZONE               │
│  TEXT                    TEXT                                   │
│  BLOB                    BYTEA                                  │
│  ENUM                    VARCHAR + CHECK or CREATE TYPE         │
│  SET                     ARRAY or separate table                │
│  JSON                    JSONB                                  │
│  AUTO_INCREMENT          SERIAL / IDENTITY                      │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│              SQL to MongoDB Mapping                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  SQL Concept             MongoDB Equivalent                     │
│  ─────────────────────────────────────────────────────────────  │
│  Database                Database                               │
│  Table                   Collection                             │
│  Row                     Document                               │
│  Column                  Field                                  │
│  Primary Key             _id field                              │
│  Foreign Key             Reference or Embedding                 │
│  JOIN                    $lookup or Embedding                   │
│  Index                   Index                                  │
│  GROUP BY                Aggregation Pipeline                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Migration Tools

```
┌─────────────────────────────────────────────────────────────────┐
│                  Migration Tools                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  SCHEMA CONVERSION                                              │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • AWS Schema Conversion Tool (SCT)                         │ │
│  │ • pgLoader (MySQL/SQLite → PostgreSQL)                     │ │
│  │ • ora2pg (Oracle → PostgreSQL)                             │ │
│  │ • SQLines (various databases)                              │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  DATA MIGRATION                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • AWS Database Migration Service (DMS)                     │ │
│  │ • GCP Database Migration Service                           │ │
│  │ • Debezium (CDC-based)                                     │ │
│  │ • Airbyte (ELT platform)                                   │ │
│  │ • pg_dump / pg_restore                                     │ │
│  │ • mysqldump / mysqlimport                                  │ │
│  │ • mongodump / mongorestore                                 │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  VALIDATION                                                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Data validation queries                                  │ │
│  │ • Row count comparisons                                    │ │
│  │ • Checksum verification                                    │ │
│  │ • Application-level testing                                │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Risk Mitigation

```
┌─────────────────────────────────────────────────────────────────┐
│                  Risk Mitigation Strategies                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  BEFORE MIGRATION                                               │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Complete backup of source database                       │ │
│  │ • Document current performance baselines                   │ │
│  │ • Test migration on non-production first                   │ │
│  │ • Create detailed rollback plan                            │ │
│  │ • Define go/no-go criteria                                 │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  DURING MIGRATION                                               │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Monitor source and target closely                        │ │
│  │ • Track sync lag in CDC migrations                         │ │
│  │ • Have rollback scripts ready                              │ │
│  │ • Communicate status to stakeholders                       │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  AFTER MIGRATION                                                │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Keep source database available (read-only)               │ │
│  │ • Monitor target performance                               │ │
│  │ • Compare query performance                                │ │
│  │ • Watch for data anomalies                                 │ │
│  │ • Plan decommission after bake period                      │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  COMMON PITFALLS                                                │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ ✗ Underestimating time for stored procedure conversion    │ │
│  │ ✗ Ignoring timezone differences                           │ │
│  │ ✗ Forgetting about triggers and events                    │ │
│  │ ✗ Not testing with production-size data                   │ │
│  │ ✗ Insufficient rollback planning                          │ │
│  │ ✗ Rushing the cutover                                     │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```
