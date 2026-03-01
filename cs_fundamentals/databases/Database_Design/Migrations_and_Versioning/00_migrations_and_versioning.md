# Migrations and Versioning

## Introduction

Database migrations are version-controlled changes to your database schema. They enable teams to evolve the database schema safely, consistently, and in sync with application code.

## Topics in This Section

1. **[Schema Migration Strategies](01_schema_migration_strategies.md)**
2. **[Zero-Downtime Migrations](02_zero_downtime_migrations.md)**
3. **[Migration Tools](03_migration_tools.md)**
4. **[Data Migration Patterns](04_data_migration_patterns.md)**

## Why Migrations Matter

```
┌─────────────────────────────────────────────────────────────────┐
│              Benefits of Migration Systems                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  VERSION CONTROL                                                │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Schema changes tracked in git                            │ │
│  │ • History of all changes                                   │ │
│  │ • Rollback capability                                      │ │
│  │ • Code review for schema changes                           │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  CONSISTENCY                                                    │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Same schema across all environments                      │ │
│  │ • Reproducible database setup                              │ │
│  │ • New developers get correct schema                        │ │
│  │ • CI/CD integration                                        │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  COLLABORATION                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Multiple developers can work on schema                   │ │
│  │ • Merge conflicts are explicit                             │ │
│  │ • Clear ownership of changes                               │ │
│  │ • Audit trail                                              │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  DEPLOYMENT                                                     │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Automated schema updates                                 │ │
│  │ • Paired with application deployments                      │ │
│  │ • Rollback together with code                              │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Migration Lifecycle

```
┌─────────────────────────────────────────────────────────────────┐
│              Migration Workflow                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ 1. CREATE MIGRATION                                      │    │
│  │    Developer creates migration file                      │    │
│  │    $ rails generate migration AddPhoneToUsers            │    │
│  └────────────────────────────┬────────────────────────────┘    │
│                               │                                  │
│                               ▼                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ 2. WRITE MIGRATION                                       │    │
│  │    Define up() and down() operations                     │    │
│  │    - up: Apply changes                                   │    │
│  │    - down: Rollback changes                              │    │
│  └────────────────────────────┬────────────────────────────┘    │
│                               │                                  │
│                               ▼                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ 3. TEST LOCALLY                                          │    │
│  │    Run migration on local database                       │    │
│  │    Verify application works                              │    │
│  │    Test rollback                                         │    │
│  └────────────────────────────┬────────────────────────────┘    │
│                               │                                  │
│                               ▼                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ 4. CODE REVIEW                                           │    │
│  │    Review migration in pull request                      │    │
│  │    Check for safety issues                               │    │
│  └────────────────────────────┬────────────────────────────┘    │
│                               │                                  │
│                               ▼                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ 5. DEPLOY TO STAGING                                     │    │
│  │    Test with production-like data                        │    │
│  │    Measure migration time                                │    │
│  └────────────────────────────┬────────────────────────────┘    │
│                               │                                  │
│                               ▼                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ 6. DEPLOY TO PRODUCTION                                  │    │
│  │    Run migration                                         │    │
│  │    Monitor for issues                                    │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

## Migration Naming Conventions

```
┌─────────────────────────────────────────────────────────────────┐
│              Naming Migrations                                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  TIMESTAMP PREFIX (Most Common)                                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ 20240115120000_create_users_table.sql                      │ │
│  │ 20240115130000_add_email_index_to_users.sql                │ │
│  │ 20240116090000_create_orders_table.sql                     │ │
│  │                                                             │ │
│  │ Benefits:                                                   │ │
│  │ • Natural ordering                                         │ │
│  │ • Avoids conflicts                                         │ │
│  │ • Clear creation time                                      │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  SEQUENTIAL NUMBERING                                           │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ V001__create_users_table.sql                               │ │
│  │ V002__add_email_index.sql                                  │ │
│  │ V003__create_orders_table.sql                              │ │
│  │                                                             │ │
│  │ Benefits:                                                   │ │
│  │ • Simple to read                                           │ │
│  │ • Clear sequence                                           │ │
│  │                                                             │ │
│  │ Drawbacks:                                                  │ │
│  │ • Can conflict in teams                                    │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  DESCRIPTIVE NAMES                                              │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Good:                                                       │ │
│  │ • add_phone_number_to_users                                │ │
│  │ • create_orders_table                                      │ │
│  │ • add_index_on_users_email                                 │ │
│  │                                                             │ │
│  │ Bad:                                                        │ │
│  │ • update_schema                                            │ │
│  │ • fix_bug                                                  │ │
│  │ • changes                                                  │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```
