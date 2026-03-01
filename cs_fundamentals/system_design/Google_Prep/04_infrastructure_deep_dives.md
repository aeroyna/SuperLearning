# Infrastructure Deep Dives

> Deep exploration of Google's core infrastructure systems that power everything else.

## Overview

Understanding Google's infrastructure gives you a significant advantage in interviews. These systems represent solutions to fundamental distributed systems problems.

---

## 1. Colossus (GFS Successor)

### What It Is

Colossus is Google's next-generation distributed file system, successor to GFS (Google File System). It stores all of Google's data: search index, YouTube videos, Gmail, etc.

### Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         Colossus Architecture                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│                    ┌─────────────────────────────────────┐                  │
│                    │           Colossus Client           │                  │
│                    │    (Library linked into apps)       │                  │
│                    └─────────────────┬───────────────────┘                  │
│                                      │                                       │
│          ┌───────────────────────────┼───────────────────────────┐          │
│          │                           │                           │          │
│          ▼                           ▼                           ▼          │
│   ┌─────────────────┐       ┌─────────────────┐        ┌─────────────────┐ │
│   │   Curator       │       │   Curator       │        │   Curator       │ │
│   │   (Metadata)    │       │   (Metadata)    │        │   (Metadata)    │ │
│   │                 │       │                 │        │                 │ │
│   │ • File→chunks   │       │ • Namespace     │        │ • Garbage       │ │
│   │ • Chunk→servers │       │   operations    │        │   collection    │ │
│   └────────┬────────┘       └────────┬────────┘        └────────┬────────┘ │
│            │                         │                          │           │
│            │                         │                          │           │
│            ▼                         ▼                          ▼           │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                         Custodians                                   │  │
│   │                   (Control plane daemons)                            │  │
│   │                                                                      │  │
│   │   • Disk health monitoring        • Replication management          │  │
│   │   • Background rebalancing        • Encoding (Reed-Solomon)         │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                      │                                       │
│            ┌─────────────────────────┼─────────────────────────┐            │
│            ▼                         ▼                         ▼            │
│   ┌─────────────────┐       ┌─────────────────┐       ┌─────────────────┐  │
│   │  D (Disk Server)│       │  D (Disk Server)│       │  D (Disk Server)│  │
│   │                 │       │                 │       │                 │  │
│   │  [Disk][Disk]   │       │  [Disk][Disk]   │       │  [Disk][Disk]   │  │
│   │  [Disk][Disk]   │       │  [Disk][Disk]   │       │  [Disk][Disk]   │  │
│   └─────────────────┘       └─────────────────┘       └─────────────────┘  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Key Improvements Over GFS

| Aspect | GFS | Colossus |
|--------|-----|----------|
| **Metadata** | Single Master | Distributed Curators |
| **Chunk Size** | 64MB | 1MB (D chunks) |
| **Encoding** | 3x replication | Reed-Solomon (1.5x) |
| **Scalability** | Limited by Master | Virtually unlimited |
| **Latency** | Higher | Lower (smaller chunks) |

### Reed-Solomon Encoding

```
┌─────────────────────────────────────────────────────────────────┐
│                    Reed-Solomon Encoding                         │
│                                                                  │
│   Original Data: 6 chunks (A, B, C, D, E, F)                    │
│                                                                  │
│   ┌───┬───┬───┬───┬───┬───┐                                     │
│   │ A │ B │ C │ D │ E │ F │  Data chunks                        │
│   └───┴───┴───┴───┴───┴───┘                                     │
│            │                                                     │
│            ▼  Encode                                            │
│   ┌───┬───┬───┬───┬───┬───┬───┬───┬───┐                         │
│   │ A │ B │ C │ D │ E │ F │ P1│ P2│ P3│                         │
│   └───┴───┴───┴───┴───┴───┴───┴───┴───┘                         │
│                           └─────┬─────┘                          │
│                           Parity chunks                          │
│                                                                  │
│   Storage overhead: 9/6 = 1.5x (vs 3x for triple replication)   │
│   Can tolerate: Any 3 chunk failures                             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### When to Mention

- Discussing storage layer design
- Data durability requirements
- Storage efficiency optimizations
- Large-scale file systems

---

## 2. Chubby - Distributed Lock Service

### What It Is

Chubby is a distributed lock service that provides coarse-grained locking and reliable storage for small files. It's the foundation for many Google systems.

### Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          Chubby Architecture                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ┌───────────────────────────────────────────────────────────────────────┐ │
│   │                         Chubby Cell                                    │ │
│   │                    (5 replicas for fault tolerance)                   │ │
│   │                                                                        │ │
│   │      ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐          │ │
│   │      │ Replica  │  │ Replica  │  │ Replica  │  │ Replica  │          │ │
│   │      │    1     │  │    2     │  │    3     │  │    4     │          │ │
│   │      │          │  │ (Leader) │  │          │  │          │  ...     │ │
│   │      └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘          │ │
│   │           │             │             │             │                 │ │
│   │           └─────────────┴──────┬──────┴─────────────┘                 │ │
│   │                                │                                       │ │
│   │                         Paxos Consensus                                │ │
│   │                                                                        │ │
│   └───────────────────────────────────────────────────────────────────────┘ │
│                                    │                                         │
│                   ┌────────────────┼────────────────┐                       │
│                   │                │                │                        │
│                   ▼                ▼                ▼                        │
│            ┌───────────┐    ┌───────────┐    ┌───────────┐                  │
│            │  Client   │    │  Client   │    │  Client   │                  │
│            │  Library  │    │  Library  │    │  Library  │                  │
│            │           │    │           │    │           │                  │
│            │  • Cache  │    │  • Cache  │    │  • Cache  │                  │
│            │  • Session│    │  • Session│    │  • Session│                  │
│            │  • Leases │    │  • Leases │    │  • Leases │                  │
│            └───────────┘    └───────────┘    └───────────┘                  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Lock Types and Usage

```python
class ChubbyClient:
    def acquire_lock(self, path: str, mode: LockMode) -> Lock:
        """
        Acquire a lock on a Chubby file/directory.

        Modes:
        - EXCLUSIVE: Only one holder
        - SHARED: Multiple readers allowed
        """
        handle = self.open(path, mode)
        handle.acquire()
        return Lock(handle)

    def leader_election(self, election_path: str) -> bool:
        """
        Participate in leader election.
        Returns True if this process becomes leader.
        """
        try:
            # Try to create ephemeral file
            handle = self.open(
                election_path,
                create=True,
                ephemeral=True
            )
            handle.acquire(EXCLUSIVE)
            return True  # We are the leader
        except AlreadyExistsException:
            # Someone else is leader
            return False
```

### Use Cases

```
┌─────────────────────────────────────────────────────────────────┐
│                     Chubby Use Cases                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. Leader Election                                              │
│     ┌─────────┐                                                  │
│     │ Process │ ──acquire(/service/leader)──▶ ┌────────┐        │
│     │    A    │                               │ Chubby │        │
│     └─────────┘                               │        │        │
│     ┌─────────┐                               │  🔒    │        │
│     │ Process │ ──acquire(/service/leader)──▶ │        │        │
│     │    B    │              (blocked)        └────────┘        │
│     └─────────┘                                                  │
│                                                                  │
│  2. Service Discovery                                            │
│     Services register themselves:                                │
│     /services/bigtable/tablets/tablet-001 → "host1:port"        │
│     /services/bigtable/tablets/tablet-002 → "host2:port"        │
│                                                                  │
│  3. Configuration Storage                                        │
│     /config/my-service/settings → {json configuration}          │
│     Clients watch for changes and get notifications             │
│                                                                  │
│  4. Access Control Lists                                         │
│     Store ACLs that control access to other systems             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Comparison with ZooKeeper

| Feature | Chubby | ZooKeeper |
|---------|--------|-----------|
| **Protocol** | Paxos | ZAB (Zookeeper Atomic Broadcast) |
| **Lock Granularity** | Coarse-grained | Fine-grained |
| **Caching** | Aggressive client caching | Watch-based |
| **Typical Use** | Google internal | Open source ecosystem |

---

## 3. Zanzibar - Global Authorization

### What It Is

Zanzibar is Google's global authorization system. It checks whether a user can perform an action on an object, handling trillions of access checks per day.

### Data Model

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      Zanzibar Relation Tuples                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Format: <object>#<relation>@<user>                                        │
│                                                                              │
│   Examples:                                                                  │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                                                                      │  │
│   │   doc:readme#owner@user:alice                                        │  │
│   │   └──┬────┘ └──┬─┘  └───┬────┘                                      │  │
│   │      │        │        │                                             │  │
│   │   object   relation  subject                                         │  │
│   │                                                                      │  │
│   │   doc:readme#viewer@user:bob                                         │  │
│   │   doc:readme#viewer@group:engineering#member                        │  │
│   │                      └──────────┬─────────┘                          │  │
│   │                        subject is a userset                          │  │
│   │                                                                      │  │
│   │   folder:root#parent@doc:readme                                     │  │
│   │   (readme is in folder root)                                         │  │
│   │                                                                      │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        Zanzibar Architecture                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                          Check Request                               │  │
│   │         "Can user:alice view doc:secret?"                           │  │
│   └────────────────────────────────┬────────────────────────────────────┘  │
│                                    │                                        │
│                                    ▼                                        │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                          ACL Server                                  │  │
│   │                                                                      │  │
│   │   1. Parse request                                                   │  │
│   │   2. Look up namespace config (schema)                               │  │
│   │   3. Evaluate permission                                             │  │
│   │   4. Return result                                                   │  │
│   └────────────────────────────────┬────────────────────────────────────┘  │
│                                    │                                        │
│          ┌─────────────────────────┴─────────────────────────┐             │
│          ▼                                                   ▼              │
│   ┌─────────────────┐                              ┌─────────────────────┐ │
│   │  Tuple Store    │                              │   Namespace Config   │ │
│   │   (Spanner)     │                              │                      │ │
│   │                 │                              │  name: "doc"        │ │
│   │ doc:secret      │                              │  relations:          │ │
│   │   #owner@alice  │                              │    owner:            │ │
│   │   #viewer@group │                              │    viewer:           │ │
│   │     :team#member│                              │      - owner        │ │
│   │                 │                              │      - parent.viewer│ │
│   └─────────────────┘                              └─────────────────────┘ │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Permission Evaluation

```python
class Zanzibar:
    def check(self, user: str, relation: str, object: str) -> bool:
        """
        Check if user has relation to object.
        Recursively expands usersets and computed relations.
        """
        # Direct check
        if self.tuple_exists(object, relation, user):
            return True

        # Get namespace config for computed relations
        config = self.get_namespace_config(object.type)

        for rule in config.relations[relation].rewrite:
            if rule.type == "computed_userset":
                # e.g., viewer includes owner
                if self.check(user, rule.relation, object):
                    return True

            elif rule.type == "tuple_to_userset":
                # e.g., viewer includes parent folder's viewers
                for parent in self.get_related(object, rule.tupleset):
                    if self.check(user, rule.computed_userset, parent):
                        return True

        return False
```

### Consistency Model

```
┌─────────────────────────────────────────────────────────────────┐
│                   Zanzibar Consistency                           │
│                                                                  │
│   Problem: "New Enemy" problem                                   │
│                                                                  │
│   Timeline:                                                      │
│   ────────────────────────────────────────────────────────────  │
│   t1: Alice shares doc with Bob                                  │
│   t2: Check: "Can Bob view doc?" → YES (on replica A)           │
│   t3: Alice removes Bob's access                                 │
│   t4: Check: "Can Bob view doc?" → YES (stale replica B!) ❌    │
│                                                                  │
│   Solution: Zookies (Zanzibar cookies)                          │
│                                                                  │
│   ┌──────────────────────────────────────────────────────────┐  │
│   │  Zookie = cryptographically signed timestamp              │  │
│   │                                                           │  │
│   │  1. Write returns zookie with timestamp T                 │  │
│   │  2. Client includes zookie in subsequent checks           │  │
│   │  3. ACL server ensures it reads data at least as fresh    │  │
│   │     as timestamp T                                        │  │
│   └──────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Scale

- **Checks**: > 10 million QPS
- **Latency**: p50 < 10ms, p99 < 100ms
- **Tuples**: Trillions
- **Storage**: Spanner (globally consistent)

---

## 4. Spanner Internals

### TrueTime Deep Dive

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           TrueTime System                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Each Data Center:                                                          │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                                                                      │  │
│   │   ┌───────────────┐     ┌───────────────┐                           │  │
│   │   │  GPS Receiver │     │ Atomic Clock  │   Multiple time sources   │  │
│   │   │    (Armageddon│     │   (Armageddon │   for redundancy          │  │
│   │   │     master)   │     │    master)    │                           │  │
│   │   └───────┬───────┘     └───────┬───────┘                           │  │
│   │           │                     │                                    │  │
│   │           └──────────┬──────────┘                                    │  │
│   │                      ▼                                               │  │
│   │              ┌───────────────┐                                       │  │
│   │              │  Time Master  │                                       │  │
│   │              │   Daemon      │                                       │  │
│   │              └───────┬───────┘                                       │  │
│   │                      │                                               │  │
│   │         ┌────────────┼────────────┐                                  │  │
│   │         ▼            ▼            ▼                                  │  │
│   │   ┌──────────┐ ┌──────────┐ ┌──────────┐                            │  │
│   │   │ Spanserver│ │ Spanserver│ │ Spanserver│                          │  │
│   │   │  TT.now() │ │  TT.now() │ │  TT.now() │                          │  │
│   │   └──────────┘ └──────────┘ └──────────┘                            │  │
│   │                                                                      │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│   TT.now() returns interval [earliest, latest]                              │
│                                                                              │
│   Time ──────────────────────────────────────────────────────────────────▶ │
│         │◄─────── ε (uncertainty) ────────▶│                               │
│         earliest                          latest                            │
│                                                                              │
│   Typical ε: 1-7ms (average 4ms)                                            │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Commit Wait

```python
class SpannerTransaction:
    def commit(self):
        """
        External consistency via commit wait.
        """
        # 1. Get commit timestamp
        s = TrueTime.now()
        commit_ts = s.latest  # Use latest possible time

        # 2. Wait until we're certain this timestamp is in the past
        # This ensures any later transaction sees our writes
        while TrueTime.now().earliest < commit_ts:
            sleep(1ms)

        # 3. Now safe to report commit
        # Any transaction starting after this point
        # will have timestamp > commit_ts
        return commit_ts
```

### Data Organization

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      Spanner Data Hierarchy                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Universe (Global Spanner deployment)                                       │
│   │                                                                          │
│   ├── Zone A (Data center region)                                           │
│   │   ├── Spanserver 1                                                       │
│   │   │   ├── Tablet 1                                                       │
│   │   │   │   ├── Directory 1 (unit of data placement)                      │
│   │   │   │   │   └── Key-value pairs                                        │
│   │   │   │   └── Directory 2                                                │
│   │   │   └── Tablet 2                                                       │
│   │   └── Spanserver 2                                                       │
│   │                                                                          │
│   ├── Zone B                                                                 │
│   │   └── ... (replicas of same tablets)                                    │
│   │                                                                          │
│   └── Zone C                                                                 │
│       └── ...                                                                │
│                                                                              │
│   Replication: Paxos groups per tablet                                       │
│   Each tablet has 3-5 replicas across zones                                  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Schema Flexibility

```sql
-- Spanner supports interleaved tables for locality
CREATE TABLE Users (
    user_id INT64 NOT NULL,
    name STRING(100),
) PRIMARY KEY (user_id);

CREATE TABLE Orders (
    user_id INT64 NOT NULL,
    order_id INT64 NOT NULL,
    amount FLOAT64,
) PRIMARY KEY (user_id, order_id),
  INTERLEAVE IN PARENT Users ON DELETE CASCADE;

-- Orders for a user are colocated with the user row
-- Enables efficient joins without distributed queries
```

---

## 5. Dremel & BigQuery

### Dremel Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         Dremel Query Execution                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│                         ┌─────────────────┐                                 │
│                         │   Root Server   │                                 │
│                         │  (Coordinator)  │                                 │
│                         └────────┬────────┘                                 │
│                                  │                                           │
│            ┌─────────────────────┼─────────────────────┐                    │
│            ▼                     ▼                     ▼                    │
│     ┌─────────────┐      ┌─────────────┐       ┌─────────────┐             │
│     │Intermediate │      │Intermediate │       │Intermediate │             │
│     │   Server    │      │   Server    │       │   Server    │             │
│     └──────┬──────┘      └──────┬──────┘       └──────┬──────┘             │
│            │                    │                     │                     │
│     ┌──────┴──────┐      ┌──────┴──────┐       ┌──────┴──────┐             │
│     ▼      ▼      ▼      ▼      ▼      ▼       ▼      ▼      ▼             │
│   ┌───┐  ┌───┐  ┌───┐  ┌───┐  ┌───┐  ┌───┐   ┌───┐  ┌───┐  ┌───┐         │
│   │Leaf│ │Leaf│ │Leaf│ │Leaf│ │Leaf│ │Leaf│  │Leaf│ │Leaf│ │Leaf│         │
│   └─┬─┘  └─┬─┘  └─┬─┘  └─┬─┘  └─┬─┘  └─┬─┘   └─┬─┘  └─┬─┘  └─┬─┘         │
│     │      │      │      │      │      │       │      │      │            │
│     ▼      ▼      ▼      ▼      ▼      ▼       ▼      ▼      ▼            │
│   ┌─────────────────────────────────────────────────────────────┐         │
│   │                    Columnar Storage                          │         │
│   │                     (Colossus)                               │         │
│   └─────────────────────────────────────────────────────────────┘         │
│                                                                              │
│   Query: SELECT country, SUM(revenue) FROM sales GROUP BY country          │
│                                                                              │
│   Execution:                                                                 │
│   1. Root parses and optimizes query                                        │
│   2. Intermediate servers manage subtrees                                    │
│   3. Leaf servers read columnar data, apply predicates                      │
│   4. Partial aggregates flow up the tree                                    │
│   5. Root produces final result                                              │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Columnar Storage Format

```
┌─────────────────────────────────────────────────────────────────┐
│                  Columnar vs Row Storage                         │
│                                                                  │
│   Row Storage (Traditional):                                     │
│   ┌──────────────────────────────────────────────────────────┐  │
│   │ Row1: [id:1, name:"Alice", age:30, city:"NYC"]          │  │
│   │ Row2: [id:2, name:"Bob", age:25, city:"LA"]             │  │
│   │ Row3: [id:3, name:"Carol", age:35, city:"NYC"]          │  │
│   └──────────────────────────────────────────────────────────┘  │
│                                                                  │
│   Columnar Storage (Dremel):                                     │
│   ┌────────────┬────────────────────────┬─────────────────────┐ │
│   │ id column  │ name column            │ city column         │ │
│   │ [1, 2, 3]  │ ["Alice","Bob","Carol"]│ ["NYC","LA","NYC"]  │ │
│   └────────────┴────────────────────────┴─────────────────────┘ │
│                                                                  │
│   Benefits:                                                      │
│   • Only read columns needed for query                          │
│   • Better compression (similar values together)                 │
│   • Vectorized processing                                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Quick Reference Table

| System | Purpose | Key Innovation | Interview Mention |
|--------|---------|----------------|-------------------|
| **Colossus** | File storage | Distributed metadata, erasure coding | Storage layer discussions |
| **Chubby** | Coordination | Coarse-grained locks, Paxos | Leader election, config storage |
| **Zanzibar** | Authorization | Relationship tuples, zookies | Access control design |
| **Spanner** | Database | TrueTime, external consistency | Global transactions |
| **Dremel** | Analytics | Columnar storage, tree execution | OLAP, BigQuery mentions |

---

## Related Topics

- [[00_google_overview|Google Prep Overview]]
- [[02_google_topics|Google-Specific Topics]]
- [[03_google_case_studies|Google Case Studies]]

---

**Tags**: #google #infrastructure #colossus #chubby #zanzibar #spanner #dremel #deep-dive
