# SQL Basics

## Overview

SQL (Structured Query Language) is the standard language for interacting with relational databases. This section covers the fundamentals from basic queries to advanced features like window functions.

## Topics Covered

1. **[DDL Data Definition Language](01_ddl_data_definition_language.md)** - CREATE, ALTER, DROP
2. **[DML Data Manipulation Language](02_dml_data_manipulation_language.md)** - INSERT, UPDATE, DELETE
3. **[DQL Data Query Language](03_dql_data_query_language.md)** - SELECT and filtering
4. **[DCL and TCL](04_dcl_and_tcl.md)** - Permissions and transaction control
5. **[Joins Deep Dive](05_joins_deep_dive.md)** - All join types explained
6. **[Subqueries and CTEs](06_subqueries_and_ctes.md)** - Nested queries and common table expressions
7. **[Aggregate Functions and Grouping](07_aggregate_functions_and_grouping.md)** - COUNT, SUM, GROUP BY
8. **[Window Functions](08_window_functions.md)** - ROW_NUMBER, RANK, LAG, LEAD

## SQL Command Categories

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         SQL COMMAND CATEGORIES                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   DDL (Data Definition)           DML (Data Manipulation)                   │
│   ─────────────────────           ───────────────────────                   │
│   CREATE TABLE                    INSERT INTO                               │
│   ALTER TABLE                     UPDATE                                    │
│   DROP TABLE                      DELETE                                    │
│   TRUNCATE TABLE                  MERGE (UPSERT)                           │
│   CREATE INDEX                                                              │
│                                                                              │
│   DQL (Data Query)                DCL (Data Control)                        │
│   ────────────────                ──────────────────                        │
│   SELECT                          GRANT                                     │
│   FROM, WHERE                     REVOKE                                    │
│   JOIN, GROUP BY                  DENY                                      │
│   ORDER BY, LIMIT                                                           │
│                                                                              │
│   TCL (Transaction Control)                                                 │
│   ─────────────────────────                                                  │
│   BEGIN / START TRANSACTION                                                 │
│   COMMIT                                                                    │
│   ROLLBACK                                                                  │
│   SAVEPOINT                                                                 │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Query Execution Order

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    SQL QUERY EXECUTION ORDER                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Written Order:              Execution Order:                               │
│   ──────────────              ────────────────                               │
│   1. SELECT                   1. FROM/JOIN    ← Start here                  │
│   2. FROM                     2. WHERE        ← Filter rows                 │
│   3. WHERE                    3. GROUP BY     ← Create groups               │
│   4. GROUP BY                 4. HAVING       ← Filter groups               │
│   5. HAVING                   5. SELECT       ← Pick columns                │
│   6. ORDER BY                 6. DISTINCT     ← Remove duplicates           │
│   7. LIMIT                    7. ORDER BY     ← Sort results                │
│                               8. LIMIT/OFFSET ← Paginate                    │
│                                                                              │
│   Example:                                                                   │
│   SELECT department, COUNT(*) as emp_count                                  │
│   FROM employees                                  -- 1. Get all employees   │
│   WHERE status = 'active'                         -- 2. Keep only active    │
│   GROUP BY department                             -- 3. Group by dept       │
│   HAVING COUNT(*) > 5                             -- 4. Keep groups > 5     │
│   ORDER BY emp_count DESC                         -- 5. Sort results        │
│   LIMIT 10;                                       -- 6. Take top 10         │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Learning Objectives

After completing this section, you will be able to:
- Write queries to create and modify database schemas
- Insert, update, and delete data efficiently
- Query data using filters, joins, and aggregations
- Use advanced features like CTEs and window functions
- Understand query execution order for optimization
- Apply SQL knowledge across different database systems
