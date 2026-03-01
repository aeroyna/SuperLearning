# Consensus Algorithms

## Introduction

Consensus algorithms enable distributed systems to agree on a single value even when some nodes fail. They are the foundation of strongly consistent distributed databases, enabling leader election, distributed locks, and replicated state machines.

## The Consensus Problem

```
┌─────────────────────────────────────────────────────────────┐
│                  Consensus Requirements                      │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  AGREEMENT: All non-faulty nodes decide on the same value  │
│  VALIDITY: The decided value was proposed by some node     │
│  TERMINATION: All non-faulty nodes eventually decide       │
│                                                              │
│  Why it's hard:                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  FLP Impossibility (1985):                          │    │
│  │  • Asynchronous system                              │    │
│  │  • Even one crash failure possible                  │    │
│  │  → No deterministic consensus algorithm exists     │    │
│  │                                                      │    │
│  │  Practical solutions:                                │    │
│  │  • Use timeouts (partial synchrony)                 │    │
│  │  • Randomization                                    │    │
│  │  • Accept liveness violations during async periods │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Failure models:                                             │
│  • Crash failures: f nodes, need 2f+1 total               │
│  • Byzantine failures: f nodes, need 3f+1 total           │
└─────────────────────────────────────────────────────────────┘
```

## Paxos

```
┌─────────────────────────────────────────────────────────────┐
│                    Paxos Algorithm                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Roles:                                                      │
│  • Proposer: Proposes values                                │
│  • Acceptor: Votes on proposals (majority needed)          │
│  • Learner: Learns decided value                            │
│                                                              │
│  Phase 1: PREPARE                                            │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Proposer → Acceptors: prepare(n)                    │    │
│  │   "I want to propose with number n"                 │    │
│  │                                                      │    │
│  │ Acceptor → Proposer: promise(n, accepted_value)    │    │
│  │   "I promise not to accept proposals < n"          │    │
│  │   "Here's the highest value I've accepted"         │    │
│  │                                                      │    │
│  │ If majority promise: proceed to Phase 2            │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Phase 2: ACCEPT                                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Proposer → Acceptors: accept(n, value)             │    │
│  │   value = highest accepted from Phase 1            │    │
│  │   or proposer's own value if none                  │    │
│  │                                                      │    │
│  │ Acceptor → Proposer: accepted(n)                   │    │
│  │   If n ≥ promised, accept and respond              │    │
│  │                                                      │    │
│  │ If majority accept: value is chosen                │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Properties:                                                 │
│  • Safety: Only one value chosen                            │
│  • Liveness: Eventually a value chosen (with luck)         │
└─────────────────────────────────────────────────────────────┘
```

### Multi-Paxos

```
┌─────────────────────────────────────────────────────────────┐
│                     Multi-Paxos                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Problem: Basic Paxos = 2 round trips per decision         │
│  Solution: Elect stable leader, skip Phase 1               │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Leader Election (Phase 1):                          │    │
│  │   Leader gets "lease" with high proposal number    │    │
│  │                                                      │    │
│  │ Normal Operation (Phase 2 only):                    │    │
│  │   Leader → Acceptors: accept(n, value)             │    │
│  │   Acceptors → Leader: accepted(n)                  │    │
│  │   1 round trip per decision!                        │    │
│  │                                                      │    │
│  │ Leader Failure:                                      │    │
│  │   New leader runs Phase 1 with higher n            │    │
│  │   Recovers any in-flight decisions                 │    │
│  │   Resumes Phase 2 only                              │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Used in: Chubby, Spanner, many production systems         │
└─────────────────────────────────────────────────────────────┘
```

## Raft

```
┌─────────────────────────────────────────────────────────────┐
│                    Raft Algorithm                            │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Designed for understandability (vs Paxos)                  │
│  Key concepts: Leader, Term, Log                            │
│                                                              │
│  Node States:                                                │
│  ┌─────────────────────────────────────────────────────┐    │
│  │                                                      │    │
│  │  ┌──────────┐  timeout  ┌───────────┐  majority     │    │
│  │  │ Follower │ ────────→ │ Candidate │ ──────────→  │    │
│  │  └──────────┘           └───────────┘               │    │
│  │       ↑                      │  │                   │    │
│  │       │                      │  │ higher term      │    │
│  │       │  higher term         │  │ discovered       │    │
│  │       └──────────────────────┘  ↓                   │    │
│  │                           ┌──────────┐              │    │
│  │                           │  Leader  │              │    │
│  │                           └──────────┘              │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Terms: Logical time periods                                │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Term 1       Term 2       Term 3       Term 4      │    │
│  │ Leader: A    Leader: B    (no leader)  Leader: C   │    │
│  │ ─────────    ─────────    ──────────   ─────────   │    │
│  │                                                      │    │
│  │ Each term has at most one leader                   │    │
│  │ Terms increase monotonically                        │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

### Raft Leader Election

```
┌─────────────────────────────────────────────────────────────┐
│                  Raft Leader Election                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Follower times out (no heartbeat from leader)          │
│  2. Increments term, becomes Candidate                      │
│  3. Votes for self, requests votes from others             │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Candidate A (term 5):                                │    │
│  │   → RequestVote(term=5, lastLogIndex, lastLogTerm) │    │
│  │                                                      │    │
│  │ Node B:                                              │    │
│  │   • If term > currentTerm: update term              │    │
│  │   • If haven't voted this term: grant vote          │    │
│  │   • If candidate's log is at least as up-to-date   │    │
│  │   ← VoteGranted(term=5)                             │    │
│  │                                                      │    │
│  │ A receives majority → becomes Leader                │    │
│  │ A sends heartbeats to maintain leadership           │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Split vote:                                                 │
│  • No candidate gets majority                               │
│  • Random timeout, try again in new term                   │
│  • Randomization prevents repeated splits                   │
└─────────────────────────────────────────────────────────────┘
```

### Raft Log Replication

```
┌─────────────────────────────────────────────────────────────┐
│                  Raft Log Replication                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Leader receives client request:                            │
│  1. Append to local log (uncommitted)                       │
│  2. Send AppendEntries to followers                        │
│  3. Wait for majority acknowledgment                        │
│  4. Commit entry, apply to state machine                   │
│  5. Respond to client                                        │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ Leader Log:                                          │    │
│  │ ┌─────┬─────┬─────┬─────┬─────┐                     │    │
│  │ │ 1:A │ 1:B │ 2:C │ 3:D │ 3:E │                     │    │
│  │ └─────┴─────┴─────┴─────┴─────┘                     │    │
│  │   ↑                 ↑     ↑                          │    │
│  │   term 1        committed  uncommitted               │    │
│  │                                                      │    │
│  │ Follower 1:                                          │    │
│  │ ┌─────┬─────┬─────┬─────┐                           │    │
│  │ │ 1:A │ 1:B │ 2:C │ 3:D │  ← needs E               │    │
│  │ └─────┴─────┴─────┴─────┘                           │    │
│  │                                                      │    │
│  │ Follower 2:                                          │    │
│  │ ┌─────┬─────┬─────┐                                 │    │
│  │ │ 1:A │ 1:B │ 2:C │  ← needs D, E                  │    │
│  │ └─────┴─────┴─────┘                                 │    │
│  │                                                      │    │
│  │ Leader tracks nextIndex per follower                │    │
│  │ Sends missing entries until caught up               │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Safety: Committed entries never lost                       │
│  Log Matching: Same index+term → same command              │
└─────────────────────────────────────────────────────────────┘
```

## Comparison

```
┌─────────────────────────────────────────────────────────────┐
│                Paxos vs Raft Comparison                      │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Aspect        │ Paxos            │ Raft                    │
│  ──────────────│──────────────────│─────────────────────────│
│  Understandability│ Complex       │ Designed for clarity   │
│  Leader        │ Optional         │ Required                │
│  Log           │ Holes allowed    │ No holes                │
│  Membership    │ Complex          │ Joint consensus         │
│  Latency       │ 2 RTT (1 w/leader)│ 1 RTT                  │
│  ──────────────│──────────────────│─────────────────────────│
│                                                              │
│  In Practice:                                                │
│  • Paxos: Chubby, Spanner (variants)                       │
│  • Raft: etcd, CockroachDB, TiKV, Consul                   │
│                                                              │
│  Both provide same guarantees, Raft easier to implement    │
└─────────────────────────────────────────────────────────────┘
```

## Byzantine Fault Tolerance

```
┌─────────────────────────────────────────────────────────────┐
│              Byzantine Fault Tolerance (BFT)                 │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Byzantine failure: Node behaves arbitrarily/maliciously   │
│  • Send conflicting messages to different nodes            │
│  • Lie about its state                                      │
│  • Collude with other Byzantine nodes                      │
│                                                              │
│  Requirement: 3f + 1 nodes to tolerate f Byzantine failures│
│                                                              │
│  PBFT (Practical BFT):                                      │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ 1. PRE-PREPARE: Primary assigns sequence number    │    │
│  │    Primary → All: (seq, view, request)             │    │
│  │                                                      │    │
│  │ 2. PREPARE: Nodes broadcast prepare                 │    │
│  │    Each → All: (PREPARE, seq, view, node_id)       │    │
│  │    Wait for 2f matching prepares                    │    │
│  │                                                      │    │
│  │ 3. COMMIT: Nodes broadcast commit                   │    │
│  │    Each → All: (COMMIT, seq, view, node_id)        │    │
│  │    Wait for 2f+1 matching commits                   │    │
│  │                                                      │    │
│  │ 4. REPLY: Execute and respond to client            │    │
│  │    Client waits for f+1 matching replies           │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  Use cases:                                                  │
│  • Blockchain                                                │
│  • Financial systems with untrusted parties                 │
│  • Multi-organization systems                                │
│                                                              │
│  Cost: O(n²) messages vs O(n) for crash-only protocols    │
└─────────────────────────────────────────────────────────────┘
```

## Consensus in Databases

```
┌─────────────────────────────────────────────────────────────┐
│              Consensus in Real Databases                     │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ETCD:                                                       │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ • Raft for consensus                                │    │
│  │ • Used by Kubernetes for cluster state             │    │
│  │ • Key-value store with watches                     │    │
│  │ • Linearizable reads/writes                         │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  COCKROACHDB:                                                │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ • Raft for each range (partition)                  │    │
│  │ • Multiple Raft groups                              │    │
│  │ • Distributed transactions via 2PC + Raft          │    │
│  │ • Serializable isolation                            │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  SPANNER:                                                    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ • Paxos for each split                              │    │
│  │ • TrueTime for global ordering                      │    │
│  │ • 2PC for cross-split transactions                  │    │
│  │ • External consistency                              │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
│  TIDB/TIKV:                                                  │
│  ┌─────────────────────────────────────────────────────┐    │
│  │ • Raft for each region                              │    │
│  │ • Multi-Raft optimization                           │    │
│  │ • Percolator-based transactions                     │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

## Key Takeaways

1. **Consensus enables agreement** - Foundation of strongly consistent systems
2. **Paxos is foundational** - Proved correct, but complex
3. **Raft is practical** - Easier to understand and implement
4. **Leader optimization** - Multi-Paxos/Raft avoid 2-phase per request
5. **Byzantine adds cost** - 3f+1 nodes, O(n²) messages
6. **Real systems use consensus** - etcd, CockroachDB, Spanner all rely on it
