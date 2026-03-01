# CAP Theorem

## Introduction

The CAP theorem, proposed by Eric Brewer in 2000 and proven by Gilbert and Lynch in 2002, states that a distributed data store can only provide two of three guarantees: Consistency, Availability, and Partition Tolerance. Understanding CAP is essential for making informed decisions about distributed database design.

## The Three Properties

```mermaid
flowchart TB
    subgraph CAP["CAP Theorem Properties"]
        subgraph C["🔒 CONSISTENCY (C)"]
            C1["All nodes see same data at same time"]
            C2["Client writes X=1 to Node A<br/>Client reads from Node B<br/>Must see X=1 (not stale)"]
            C3["= Linearizability (strong consistency)"]
        end

        subgraph A["⚡ AVAILABILITY (A)"]
            A1["Every request receives a response"]
            A2["No request hangs indefinitely<br/>Non-failing nodes must respond<br/>Response must be meaningful"]
        end

        subgraph P["🔗 PARTITION TOLERANCE (P)"]
            P1["System operates despite network failures"]
            P2["Messages may be lost or delayed<br/>Network can split into groups<br/>System handles this gracefully"]
        end
    end

    C1 --> C2 --> C3
    A1 --> A2
    P1 --> P2

    style C fill:#e6f3ff
    style A fill:#e6ffe6
    style P fill:#fff0e6
```

## The Trade-off

```mermaid
flowchart TD
    subgraph Triangle["CAP Triangle"]
        C["🔒 Consistency"]
        A["⚡ Availability"]
        P["🔗 Partition Tolerance"]

        C --- CP["CP Systems"]
        C --- CA["CA Systems"]
        A --- CA
        A --- AP["AP Systems"]
        P --- CP
        P --- AP
    end

    CP --> CPEx["Refuse writes during partition<br/>Wait for partition to heal<br/>📦 HBase, MongoDB, Redis Cluster"]
    AP --> APEx["Accept writes on all partitions<br/>Reconcile conflicts later<br/>📦 Cassandra, DynamoDB, CouchDB"]
    CA --> CAEx["Only single-node or reliable network<br/>Not realistic for distributed<br/>📦 Traditional RDBMS (single node)"]

    Note["⚠️ P is reality - Networks WILL fail<br/>Real choice: CP or AP during failures"]

    style CP fill:#cce5ff
    style AP fill:#ccffcc
    style CA fill:#ffcccc
    style Note fill:#ffffcc
```

## Partition Scenarios

### CP System Behavior

```mermaid
sequenceDiagram
    participant Client
    participant A as Node A (Leader)
    participant B as Node B (Follower)
    participant C as Node C (Follower)

    Note over Client,C: 🔄 Normal Operation

    Client->>A: Write request
    A->>B: Replicate
    A->>C: Replicate
    B-->>A: ACK
    C-->>A: ACK
    A-->>Client: OK (majority ack)

    Note over Client,C: ⚡ Network Partition: {A,B} | {C}

    rect rgb(200, 255, 200)
        Note over A,B: Majority side {A, B}
        Client->>A: Write request
        A->>B: Replicate
        B-->>A: ACK (2/3 = majority ✅)
        A-->>Client: OK ✅
    end

    rect rgb(255, 200, 200)
        Note over C: Minority side {C}
        Client->>C: Read request
        C-->>Client: ERROR ❌ (no quorum)
        Note over C: Cannot reach leader<br/>Cannot form quorum<br/>Rejects all requests
    end

    Note over Client,C: ✅ Consistency guaranteed<br/>❌ Minority partition unavailable
```

### AP System Behavior

```mermaid
sequenceDiagram
    participant Client1 as Client 1
    participant A as Node A
    participant B as Node B
    participant C as Node C
    participant Client2 as Client 2

    Note over Client1,Client2: 🔄 Normal Operation

    Client1->>A: Write (any node)
    A-->>Client1: OK (local write)
    A-)B: Async replicate
    A-)C: Async replicate

    Note over Client1,Client2: ⚡ Network Partition: {A,B} | {C}

    rect rgb(200, 255, 200)
        Note over A,B: Side {A, B}
        Client1->>A: balance = 100
        A-->>Client1: OK ✅
    end

    rect rgb(200, 200, 255)
        Note over C: Side {C}
        Client2->>C: balance = 200
        C-->>Client2: OK ✅
    end

    Note over A,C: ⚠️ CONFLICT! Same user, different values

    Note over Client1,Client2: 🔧 After Partition Heals - Resolution

    rect rgb(255, 255, 200)
        Note over A,C: Conflict Resolution Options:<br/>LWW: Use timestamps<br/>Vector clocks: Detect concurrent writes<br/>App resolution: Return both to client<br/>CRDTs: Mathematically mergeable
    end

    Note over Client1,Client2: ✅ Always available<br/>❌ Possible inconsistency
```

## Real-World Examples

### CP Systems

```mermaid
flowchart TB
    subgraph CP["🔒 CP System Examples"]
        subgraph ZK["Zookeeper"]
            ZK1["Coordination service"]
            ZK2["Majority quorum for writes"]
            ZK3["Minority = read-only"]
            ZK4["📋 Leader election, config mgmt"]
        end

        subgraph ETCD["etcd"]
            E1["Distributed KV store (Raft)"]
            E2["Used by Kubernetes"]
            E3["Strong consistency"]
            E4["❌ Unavailable if no quorum"]
        end

        subgraph HB["HBase"]
            H1["Wide-column on HDFS"]
            H2["Region servers → key ranges"]
            H3["Region unavailable on failure"]
            H4["Until failover completes"]
        end

        subgraph MDB["MongoDB (w=majority)"]
            M1["Default w=1 → more AP"]
            M2["w=majority → more CP"]
            M3["Read preference: primary"]
        end
    end

    style ZK fill:#e6f3ff
    style ETCD fill:#e6f3ff
    style HB fill:#e6f3ff
    style MDB fill:#e6f3ff
```

### AP Systems

```mermaid
flowchart TB
    subgraph AP["⚡ AP System Examples"]
        subgraph CASS["Cassandra"]
            C1["Tunable consistency per query"]
            C2["CL=ONE: Very AP"]
            C3["CL=QUORUM: Balanced"]
            C4["CL=ALL: More CP"]
            C5["Hinted handoff + LWW"]
        end

        subgraph DDB["DynamoDB"]
            D1["Default: Eventually consistent"]
            D2["Optional: Strong consistency"]
            D3["Global tables (async)"]
            D4["LWW conflict resolution"]
        end

        subgraph COUCH["CouchDB"]
            CO1["Master-master replication"]
            CO2["Revision tree conflicts"]
            CO3["App chooses winner"]
            CO4["Offline-first design"]
        end

        subgraph DNS["DNS"]
            DN1["Highly available (caching)"]
            DN2["Eventually consistent (TTL)"]
            DN3["Partition tolerant"]
        end
    end

    style CASS fill:#e6ffe6
    style DDB fill:#e6ffe6
    style COUCH fill:#e6ffe6
    style DNS fill:#e6ffe6
```

## CAP Misconceptions

```mermaid
flowchart TB
    subgraph Myths["❌ Common CAP Misconceptions"]
        M1["❌ Choose 2 of 3 at design time"]
        M2["❌ CP = always unavailable"]
        M3["❌ AP = never consistent"]
        M4["❌ CAP is binary"]
        M5["❌ Partition tolerance is optional"]
        M6["❌ Latency doesn't matter"]
    end

    subgraph Truth["✅ The Reality"]
        T1["✅ Trade-off only during partition<br/>Normal: Can have C + A"]
        T2["✅ CP: Available when no partition"]
        T3["✅ AP: Eventually consistent"]
        T4["✅ Spectrum of trade-offs<br/>Tunable per operation"]
        T5["✅ Networks WILL fail<br/>P is reality, not choice"]
        T6["✅ Slow = practically unavailable<br/>See PACELC theorem"]
    end

    M1 --> T1
    M2 --> T2
    M3 --> T3
    M4 --> T4
    M5 --> T5
    M6 --> T6

    style Myths fill:#ffcccc
    style Truth fill:#ccffcc
```

## Choosing CP vs AP

```mermaid
flowchart TD
    Start{{"🤔 CP or AP?"}}

    Start --> Q1{"Is wrong data<br/>unacceptable?"}
    Q1 -->|Yes| CP["🔒 Choose CP"]
    Q1 -->|No| Q2{"Is downtime<br/>unacceptable?"}
    Q2 -->|Yes| AP["⚡ Choose AP"]
    Q2 -->|No| Hybrid["🔀 Consider Hybrid"]

    subgraph CPUse["CP Use Cases"]
        CP1["💰 Financial transactions"]
        CP2["📦 Inventory management"]
        CP3["👤 Username uniqueness"]
        CP4["🔐 Leader election, locks"]
        CP5["📋 Audit logs"]
    end

    subgraph APUse["AP Use Cases"]
        AP1["🛒 Shopping cart"]
        AP2["📱 Social media feeds"]
        AP3["📊 Analytics/logging"]
        AP4["⚡ Caching layers"]
        AP5["🌐 DNS, CDN"]
    end

    subgraph HybridUse["Hybrid Approach"]
        H1["Different guarantees per data type"]
        H2["User data: AP (always writable)"]
        H3["Financial: CP (strict)"]
        H4["Per-request consistency levels"]
    end

    CP --> CPUse
    AP --> APUse
    Hybrid --> HybridUse

    style CP fill:#cce5ff
    style AP fill:#ccffcc
    style Hybrid fill:#ffffcc
```

## Key Takeaways

1. **Partition tolerance is mandatory** - Networks fail, plan for it
2. **Trade-off is per-partition** - Normal operation can have C+A
3. **CAP is simplified** - Real systems have nuanced trade-offs
4. **Consistency has levels** - From linearizable to eventual
5. **Availability has degrees** - 99.9% vs 99.99% matters
6. **Choose based on requirements** - Money needs CP, carts can be AP
