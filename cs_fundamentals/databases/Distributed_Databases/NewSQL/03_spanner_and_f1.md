# Google Spanner and F1

## Spanner Overview

```
┌─────────────────────────────────────────────────────────────┐
│                   Google Spanner                             │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  First globally-distributed database with:                  │
│  • External consistency (linearizability)                   │
│  • SQL support                                              │
│  • Automatic sharding and replication                       │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                    Spanner Architecture              │    │
│  │                                                      │    │
│  │  Zone A          Zone B          Zone C             │    │
│  │  (US-East)       (US-West)       (Europe)          │    │
│  │  ┌─────────┐    ┌─────────┐    ┌─────────┐         │    │
│  │  │ Span-   │    │ Span-   │    │ Span-   │         │    │
│  │  │ server  │←──→│ server  │←──→│ server  │         │    │
│  │  └─────────┘    └─────────┘    └─────────┘         │    │
│  │       │              │              │               │    │
│  │       ▼              ▼              ▼               │    │
│  │  ┌─────────┐    ┌─────────┐    ┌─────────┐         │    │
│  │  │Colossus │    │Colossus │    │Colossus │         │    │
│  │  │(Storage)│    │(Storage)│    │(Storage)│         │    │
│  │  └─────────┘    └─────────┘    └─────────┘         │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Each spanserver manages 100-1000 tablets                   │
│  Tablets replicated across zones via Paxos                 │
└─────────────────────────────────────────────────────────────┘
```

## TrueTime

```
┌─────────────────────────────────────────────────────────────┐
│                     TrueTime API                             │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  The key innovation enabling global consistency            │
│                                                              │
│  TrueTime returns interval, not instant:                   │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                                                      │    │
│  │  TT.now() → [earliest, latest]                      │    │
│  │                                                      │    │
│  │  Time ─────────────────────────────────────────►    │    │
│  │           │←───────────────→│                       │    │
│  │        earliest           latest                   │    │
│  │           │    ε (epsilon)  │                       │    │
│  │           │   uncertainty   │                       │    │
│  │                                                      │    │
│  │  Actual time is guaranteed within this interval    │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Implementation:                                            │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ GPS receivers + Atomic clocks                       │    │
│  │ • GPS: Accurate to microseconds                     │    │
│  │ • Atomic: Provides backup, different failure modes │    │
│  │                                                      │    │
│  │ Time masters in each datacenter                     │    │
│  │ Servers poll multiple masters                       │    │
│  │ ε typically 1-7 milliseconds                        │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Commit Wait:                                               │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Before committing at timestamp T:                   │    │
│  │ Wait until TT.after(T) is true                      │    │
│  │                                                      │    │
│  │ Ensures T is definitely in the past                 │    │
│  │ Guarantees external consistency                     │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

## Data Model and SQL

```
┌─────────────────────────────────────────────────────────────┐
│              Spanner Data Model                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Hierarchical tables with interleaving:                    │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ CREATE TABLE Users (                                │    │
│  │   UserId INT64 NOT NULL,                            │    │
│  │   Name STRING(100),                                 │    │
│  │   Email STRING(200),                                │    │
│  │ ) PRIMARY KEY (UserId);                             │    │
│  │                                                      │    │
│  │ CREATE TABLE Orders (                               │    │
│  │   UserId INT64 NOT NULL,                            │    │
│  │   OrderId INT64 NOT NULL,                           │    │
│  │   Amount FLOAT64,                                   │    │
│  │ ) PRIMARY KEY (UserId, OrderId),                    │    │
│  │   INTERLEAVE IN PARENT Users ON DELETE CASCADE;    │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Interleaving stores child rows with parent:               │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Physical Layout:                                     │    │
│  │                                                      │    │
│  │ User 1                                               │    │
│  │   ├── Order 1                                        │    │
│  │   ├── Order 2                                        │    │
│  │   └── Order 3                                        │    │
│  │ User 2                                               │    │
│  │   ├── Order 1                                        │    │
│  │   └── Order 2                                        │    │
│  │                                                      │    │
│  │ Benefits:                                            │    │
│  │ • Parent-child joins are local                      │    │
│  │ • Related data co-located in same split            │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

## F1 Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                  F1 (Built on Spanner)                       │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  F1: Google's AdWords database (now Cloud Spanner)         │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              F1 Architecture                         │    │
│  │                                                      │    │
│  │  ┌────────────────────────────────────────────────┐ │    │
│  │  │            F1 Servers (Stateless)              │ │    │
│  │  │  ┌──────┐  ┌──────┐  ┌──────┐  ┌──────┐       │ │    │
│  │  │  │ F1   │  │ F1   │  │ F1   │  │ F1   │       │ │    │
│  │  │  └──────┘  └──────┘  └──────┘  └──────┘       │ │    │
│  │  │  SQL parsing, query optimization, execution   │ │    │
│  │  └────────────────────┬───────────────────────────┘ │    │
│  │                       │                             │    │
│  │  ┌────────────────────▼───────────────────────────┐ │    │
│  │  │              Spanner                            │ │    │
│  │  │  Distributed storage with TrueTime             │ │    │
│  │  └────────────────────────────────────────────────┘ │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  F1 Added to Spanner:                                       │
│  • Full SQL support (Spanner initially limited)            │
│  • Distributed query execution                              │
│  • Change history tracking                                  │
│  • Optimistic transactions                                  │
│                                                              │
│  Eventually merged: Cloud Spanner = Spanner + F1 features  │
└─────────────────────────────────────────────────────────────┘
```

## Transaction Types

```
┌─────────────────────────────────────────────────────────────┐
│              Spanner Transaction Types                       │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  READ-WRITE TRANSACTIONS:                                   │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ • Pessimistic locking with 2PC                      │    │
│  │ • Acquire locks during execution                    │    │
│  │ • Paxos for replication                             │    │
│  │ • Commit wait for external consistency              │    │
│  │                                                      │    │
│  │ Flow:                                                │    │
│  │ 1. Client reads/writes, acquires locks             │    │
│  │ 2. Prepare: Leader prepares, gets timestamp        │    │
│  │ 3. Commit wait: Wait until timestamp in past       │    │
│  │ 4. Commit: Apply and release locks                 │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  READ-ONLY TRANSACTIONS:                                    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ • Lock-free, uses MVCC                              │    │
│  │ • Pick timestamp, read consistent snapshot          │    │
│  │ • Can read from any sufficiently up-to-date replica│    │
│  │ • No commit wait needed                             │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  STALE READS:                                               │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ • Read at a timestamp in the past                   │    │
│  │ • Bounded staleness (e.g., 10 seconds)             │    │
│  │ • Lower latency (skip commit wait)                  │    │
│  │ • Good for analytics, dashboards                    │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

## Cloud Spanner

```
┌─────────────────────────────────────────────────────────────┐
│              Google Cloud Spanner                            │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Managed service based on internal Spanner + F1            │
│                                                              │
│  Key Features:                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ ✓ Global distribution with strong consistency      │    │
│  │ ✓ 99.999% availability SLA (regional: 99.99%)     │    │
│  │ ✓ Automatic sharding and replication               │    │
│  │ ✓ Standard SQL (GoogleSQL + PostgreSQL dialects)  │    │
│  │ ✓ ACID transactions                                 │    │
│  │ ✓ Auto-scaling                                      │    │
│  │ ✓ Point-in-time recovery                           │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Configurations:                                            │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Regional: 3+ replicas in one region                │    │
│  │ Multi-regional: Replicas across continents         │    │
│  │                                                      │    │
│  │ Pricing: Per node-hour + storage                    │    │
│  │ Min: 1 node (100 nodes max per instance)           │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Use Cases:                                                 │
│  • Global financial systems                                 │
│  • Inventory and supply chain                              │
│  • Gaming leaderboards                                      │
│  • Any app needing global ACID                             │
└─────────────────────────────────────────────────────────────┘
```
