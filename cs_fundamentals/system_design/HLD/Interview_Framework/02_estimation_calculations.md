# Back-of-Envelope Estimation

Quick calculations to understand system scale. These help you make informed design decisions.

---

## Why Estimation Matters

```
"Design Twitter"

Without estimation:
- Random database choice
- Over/under-provisioned infrastructure
- Missing scalability concerns

With estimation:
- 12K QPS for reads → Need caching
- 5TB/year storage → Consider sharding timeline
- 1B requests/day → CDN for static content
```

---

## Key Numbers to Memorize

### Time
```
1 day    = 86,400 seconds ≈ 100,000 seconds
1 month  = 2.5 million seconds
1 year   = 31 million seconds ≈ 30 million seconds

Quick conversion:
requests/day ÷ 100,000 ≈ requests/second
```

### Data Size
```
1 byte   = 8 bits
1 char   = 1-4 bytes (UTF-8)
1 KB     = 1,000 bytes
1 MB     = 1,000 KB = 10^6 bytes
1 GB     = 1,000 MB = 10^9 bytes
1 TB     = 1,000 GB = 10^12 bytes
1 PB     = 1,000 TB = 10^15 bytes

UUID    = 36 chars = 36 bytes
Timestamp = 8 bytes
Integer = 4-8 bytes
```

### Latency
```
L1 cache reference:           0.5 ns
L2 cache reference:           7 ns
RAM reference:                100 ns
SSD random read:              150 μs
HDD random read:              10 ms
Same datacenter round trip:   0.5 ms
Cross-continent round trip:   150 ms
```

### Throughput
```
HDD sequential read:   100 MB/s
SSD sequential read:   500 MB/s
1 Gbps network:        125 MB/s
10 Gbps network:       1.25 GB/s

Redis:                 100K+ ops/sec
PostgreSQL:            10K-50K queries/sec
```

---

## Estimation Framework

### Step 1: Traffic Estimation

```
Given: 100M DAU, each user does 10 actions/day

Daily requests = 100M × 10 = 1B requests/day
QPS = 1B / 100,000 = 10,000 QPS

Peak QPS = 2-3x average = 20,000-30,000 QPS
```

### Step 2: Storage Estimation

```
Each record = 500 bytes
100M records/day = 50 GB/day

1 year = 50 GB × 365 = 18 TB/year
5 years = 90 TB

With 3x replication = 270 TB
```

### Step 3: Bandwidth Estimation

```
Incoming (writes):
100M records × 500 bytes = 50 GB/day
50 GB / 86,400 = 600 KB/s

Outgoing (reads):
1B reads × 500 bytes = 500 GB/day
500 GB / 86,400 = 6 MB/s
```

### Step 4: Memory Estimation (Cache)

```
Cache top 20% of data (Pareto principle)
Total data: 100 GB
Cache size: 20 GB

Or cache last N hours of data:
6 hours of data at 50 GB/day = 12.5 GB
```

---

## Common Estimation Scenarios

### Twitter/Social Media

```
DAU: 100M users
Posts/day: 50M (0.5 per user)
Read:Write ratio: 100:1

Traffic:
- Writes: 50M / 100,000 = 500 QPS
- Reads: 5B / 100,000 = 50,000 QPS

Storage (1 year):
- 50M posts × 300 bytes = 15 GB/day
- 15 GB × 365 = 5.5 TB/year

Conclusion:
- Heavy read → Need aggressive caching
- Moderate storage → Sharding can wait
```

### Chat Application

```
DAU: 100M users
Messages/user/day: 50

Traffic:
- 5B messages/day
- 5B / 100,000 = 50,000 messages/sec
- Peak: 150,000 messages/sec

Storage (1 year):
- 5B × 100 bytes = 500 GB/day
- 500 GB × 365 = 180 TB/year

Conclusion:
- Very high throughput → Message queue essential
- Large storage → Partition by conversation/user
```

### Video Streaming

```
DAU: 50M users
Average watch time: 30 min/day

Bandwidth:
- Video bitrate: 5 Mbps (HD)
- 50M × 30 min × 5 Mbps = huge!
- Actually: 50M × 1800s × 5 Mb / 8 = 56 PB/day

Conclusion:
- Need CDN globally
- Adaptive bitrate streaming
- Pre-encoding multiple qualities
```

---

## Quick Estimation Cheat Sheet

| What | Formula |
|------|---------|
| QPS | total_requests / 100,000 |
| Peak QPS | avg_QPS × 2-3 |
| Storage/year | daily_data × 365 |
| Cache size | total_data × 0.2 |
| Servers needed | QPS / server_capacity |

---

## Presentation Tips

```
"Let me do some quick math.

We have 100M DAU, each reading feed 10 times/day.
That's 1 billion reads per day.

Dividing by roughly 100K seconds per day...
That's about 10,000 QPS for reads.

At peak, maybe 2-3x, so let's plan for 30K QPS.

Does this order of magnitude seem right to you?"
```

**Key points:**
- Round aggressively (100K instead of 86,400)
- State assumptions clearly
- Check in with interviewer
- Focus on order of magnitude, not exact numbers

---

## Practice Problems

1. **Instagram**: 500M DAU, 100M photos uploaded/day, each photo 2MB
   - Storage per year?
   - Bandwidth for uploads?

2. **URL Shortener**: 100M new URLs/month, 100:1 read ratio
   - QPS for reads and writes?
   - Storage for 5 years?

3. **Uber**: 10M rides/day, location update every 5 seconds
   - QPS for location updates?
   - Storage for trip history (1 year)?

---

## Solutions

### Instagram
```
Storage: 100M × 2MB × 365 = 73 PB/year
Bandwidth: 100M × 2MB / 86,400 = 2.3 GB/s upload
```

### URL Shortener
```
Writes: 100M / 30 days / 86,400 = 40 QPS
Reads: 40 × 100 = 4,000 QPS
Storage: 100M × 500 bytes × 60 months = 3 TB
```

### Uber
```
Active rides: 10M / 24 hours ≈ 400K concurrent
Location updates: 400K × (1/5) = 80K QPS
Storage: 10M rides × 20 locations × 50 bytes × 365 = 3.5 TB/year
```
