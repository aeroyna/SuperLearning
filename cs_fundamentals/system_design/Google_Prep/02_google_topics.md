# Google-Specific Topics

> Key technologies and concepts from Google's infrastructure that are valuable for interviews.

## Overview

Google has published numerous papers on their infrastructure. Understanding these systems demonstrates technical depth and helps you speak Google's language during interviews.

---

## 1. Spanner - Global Distributed Database

### What It Is
Spanner is Google's globally distributed, strongly consistent database that powers critical services like Google Ads and Google Play.

### Key Concepts

```
┌─────────────────────────────────────────────────────────────────┐
│                         Spanner                                  │
├─────────────────────────────────────────────────────────────────┤
│  • Globally distributed                                          │
│  • Externally consistent (linearizable)                          │
│  • SQL support at scale                                          │
│  • Automatic sharding and replication                            │
│  • Uses TrueTime for consistency                                 │
└─────────────────────────────────────────────────────────────────┘
```

### TrueTime API

```python
# TrueTime returns an interval, not a point
class TrueTime:
    def now() -> TimeInterval:
        """Returns [earliest, latest] possible current time"""
        return TimeInterval(earliest, latest)

    def after(t: Timestamp) -> bool:
        """Returns True if t is definitely in the past"""
        pass

    def before(t: Timestamp) -> bool:
        """Returns True if t is definitely in the future"""
        pass
```

**Why TrueTime Matters:**
- GPS receivers + atomic clocks in data centers
- Bounded uncertainty (typically < 7ms)
- Enables external consistency without coordination

### When to Mention Spanner
- Global consistency requirements
- Financial transactions across regions
- Strong consistency at scale
- SQL requirements with global distribution

---

## 2. Bigtable - Wide-Column Store

### What It Is
A distributed storage system for structured data at massive scale.

### Data Model

```
┌─────────────────────────────────────────────────────────────────┐
│  Row Key  │  Column Family: content  │  Column Family: meta     │
├───────────┼─────────────────────────┼──────────────────────────┤
│  com.cnn  │  html: "<html>..."      │  title: "CNN"            │
│           │  images: [binary]       │  updated: "2024-01-15"   │
├───────────┼─────────────────────────┼──────────────────────────┤
│  com.bbc  │  html: "<html>..."      │  title: "BBC"            │
│           │  css: "body{...}"       │  language: "en"          │
└─────────────────────────────────────────────────────────────────┘

Key Properties:
- Rows sorted lexicographically by row key
- Column families defined at schema time
- Columns within family created on the fly
- Each cell can have multiple timestamped versions
```

### Architecture

```
                    ┌─────────────┐
                    │   Client    │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │   Bigtable  │
                    │   Master    │
                    └──────┬──────┘
                           │
         ┌─────────────────┼─────────────────┐
         │                 │                 │
   ┌─────▼─────┐    ┌─────▼─────┐    ┌─────▼─────┐
   │  Tablet   │    │  Tablet   │    │  Tablet   │
   │  Server   │    │  Server   │    │  Server   │
   └─────┬─────┘    └─────┬─────┘    └─────┬─────┘
         │                │                │
         └────────────────┼────────────────┘
                          │
                    ┌─────▼─────┐
                    │  Colossus │
                    │   (GFS)   │
                    └───────────┘
```

### Row Key Design (Critical!)

```python
# GOOD: Time-based data with reversed timestamp
row_key = f"{user_id}#{MAX_TIMESTAMP - timestamp}"
# Allows efficient "get latest N" queries

# BAD: Sequential timestamps
row_key = f"{timestamp}#{user_id}"
# Creates hotspots as all writes go to same tablet

# GOOD: Hashed prefix for even distribution
row_key = f"{hash(user_id) % 100}#{user_id}#{timestamp}"
```

### When to Mention Bigtable
- Time-series data
- High write throughput
- Sparse data (many optional columns)
- When you need HBase/Cassandra-like semantics

---

## 3. MapReduce & Dataflow

### MapReduce (Batch Processing)

```python
# Word Count Example

def mapper(document):
    for word in document.split():
        yield (word, 1)

def reducer(word, counts):
    yield (word, sum(counts))

# Execution:
# 1. Map phase: Process input splits in parallel
# 2. Shuffle: Group by key across machines
# 3. Reduce phase: Aggregate each key's values
```

### Dataflow (Stream + Batch)

```
┌─────────────────────────────────────────────────────────────────┐
│                    Dataflow Concepts                             │
├─────────────────────────────────────────────────────────────────┤
│  PCollection  │  Immutable distributed dataset                   │
│  PTransform   │  Operation on PCollections                       │
│  Pipeline     │  Entire data processing graph                    │
│  Windowing    │  Grouping unbounded data by time                 │
│  Triggers     │  When to emit results                            │
└─────────────────────────────────────────────────────────────────┘
```

### Windowing Strategies

```python
# Fixed Windows: Every 5 minutes
events | WindowInto(FixedWindows(5 * 60))

# Sliding Windows: 5-min windows, every 1 min
events | WindowInto(SlidingWindows(5 * 60, 1 * 60))

# Session Windows: Gap of 10 minutes ends session
events | WindowInto(Sessions(10 * 60))
```

### When to Mention
- Batch processing at scale (MapReduce)
- Real-time + batch unified (Dataflow)
- Complex event processing
- ETL pipelines

---

## 4. Borg & Kubernetes Origins

### Borg Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        Borg Cell                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                    BorgMaster                            │    │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐    │    │
│  │  │Scheduler│  │ Borglet │  │  Paxos  │  │ Master  │    │    │
│  │  │         │  │  Proxy  │  │ Store   │  │  UI     │    │    │
│  │  └─────────┘  └─────────┘  └─────────┘  └─────────┘    │    │
│  └─────────────────────────────────────────────────────────┘    │
│                              │                                   │
│         ┌────────────────────┼────────────────────┐              │
│         ▼                    ▼                    ▼              │
│  ┌─────────────┐     ┌─────────────┐      ┌─────────────┐       │
│  │   Machine   │     │   Machine   │      │   Machine   │       │
│  │  ┌───────┐  │     │  ┌───────┐  │      │  ┌───────┐  │       │
│  │  │Borglet│  │     │  │Borglet│  │      │  │Borglet│  │       │
│  │  └───────┘  │     │  └───────┘  │      │  └───────┘  │       │
│  │  [Tasks]    │     │  [Tasks]    │      │  [Tasks]    │       │
│  └─────────────┘     └─────────────┘      └─────────────┘       │
└─────────────────────────────────────────────────────────────────┘
```

### Key Concepts

| Borg Concept | Kubernetes Equivalent |
|--------------|----------------------|
| Cell | Cluster |
| Job | Deployment |
| Task | Pod |
| Alloc | PersistentVolume |
| BorgMaster | API Server + Controller Manager |
| Borglet | Kubelet |

### Scheduling Priorities

```
Priority Classes (high to low):
1. Monitoring (always runs)
2. Production (guaranteed resources)
3. Batch (best effort)
4. Free tier (preemptible)
```

### When to Mention
- Container orchestration discussions
- Resource scheduling strategies
- Cluster management at scale
- Understanding Kubernetes design decisions

---

## 5. Pub/Sub - Messaging at Scale

### Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      Cloud Pub/Sub                               │
│                                                                  │
│   Publishers ───▶ ┌─────────┐ ───▶ Subscriptions ───▶ Subscribers│
│                   │  Topic  │                                    │
│   Publisher 1 ──▶ │         │ ──▶ Sub A (Pull) ──▶ Subscriber 1 │
│   Publisher 2 ──▶ │         │ ──▶ Sub B (Push) ──▶ Subscriber 2 │
│                   └─────────┘ ──▶ Sub C (Pull) ──▶ Subscriber 3 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Delivery Semantics

```python
# At-least-once delivery (default)
# Messages may be delivered multiple times
# Subscribers must handle duplicates

# Ordering (optional, within ordering key)
message = {
    "data": "event_data",
    "ordering_key": "user_123"  # All messages with same key ordered
}

# Exactly-once processing (via Dataflow)
# Requires idempotent processing + deduplication
```

### Key Features

| Feature | Description |
|---------|-------------|
| **Scalability** | Millions of messages/sec |
| **Durability** | Messages stored across zones |
| **Push/Pull** | Flexible delivery modes |
| **Dead Letter** | Failed message handling |
| **Filtering** | Attribute-based filtering |
| **Ordering** | Per-key ordering guarantee |

### Message Flow

```python
# Publishing
from google.cloud import pubsub_v1

publisher = pubsub_v1.PublisherClient()
topic_path = publisher.topic_path(project, topic)

future = publisher.publish(
    topic_path,
    data=b"message",
    attribute1="value1"
)
message_id = future.result()

# Subscribing (Pull)
subscriber = pubsub_v1.SubscriberClient()
subscription_path = subscriber.subscription_path(project, subscription)

def callback(message):
    print(f"Received: {message.data}")
    message.ack()  # Acknowledge processing

subscriber.subscribe(subscription_path, callback=callback)
```

### When to Mention
- Decoupling services
- Event-driven architecture
- Real-time data pipelines
- Cross-service communication at scale

---

## Quick Comparison

| System | Use Case | Key Strength |
|--------|----------|--------------|
| **Spanner** | Global transactions | Strong consistency + SQL |
| **Bigtable** | Time-series, high write | Scale + flexible schema |
| **MapReduce** | Batch processing | Simplicity + fault tolerance |
| **Dataflow** | Stream processing | Unified batch + stream |
| **Borg** | Container orchestration | Resource efficiency |
| **Pub/Sub** | Messaging | Scalability + durability |

---

## Related Topics

- [[00_google_overview|Google Prep Overview]]
- [[04_infrastructure_deep_dives|Infrastructure Deep Dives]]
- [[03_google_case_studies|Google Case Studies]]

---

**Tags**: #google #spanner #bigtable #mapreduce #pubsub #infrastructure
