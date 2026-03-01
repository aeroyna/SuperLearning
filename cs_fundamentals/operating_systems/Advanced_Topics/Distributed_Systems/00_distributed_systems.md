# Distributed Systems

## Overview

Distributed systems involve multiple computers working together. Understanding distributed OS concepts is increasingly important.

## Topics Covered

1. **[Distributed System Architecture](01_distributed_architecture.md)**
   - Client-server model
   - Peer-to-peer model
   - Three-tier architecture
   - Microservices
   - Communication protocols

2. **[Distributed File Systems](02_distributed_file_systems.md)**
   - NFS (Network File System)
   - AFS (Andrew File System)
   - GFS (Google File System)
   - HDFS (Hadoop Distributed File System)
   - Consistency models

3. **[Distributed Coordination](03_distributed_coordination.md)**
   - Distributed mutual exclusion
   - Leader election algorithms
   - Consensus algorithms (Paxos, Raft)
   - Distributed transactions
   - Two-phase commit

4. **[Distributed Synchronization](04_distributed_synchronization.md)**
   - Clock synchronization
   - Logical clocks (Lamport timestamps)
   - Vector clocks
   - Causal ordering
   - Distributed deadlock detection

## Key Takeaways

- Distributed systems face challenges like network latency and partial failures
- Consistency, availability, and partition tolerance are trade-offs (CAP theorem)
- Consensus is hard but essential in distributed systems
- Clock synchronization is a fundamental problem

## Interview Focus

- Understand CAP theorem
- Explain consensus algorithms
- Compare NFS and other distributed file systems
- Understand clock synchronization challenges
