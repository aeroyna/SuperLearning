# Estimation Cheatsheet

Quick reference numbers for back-of-envelope calculations.

---

## Time Conversions

```
1 day      = 86,400 seconds ≈ 100,000 (10^5) seconds
1 month    = 2.5 million seconds
1 year     = 31 million seconds ≈ 30 million (3×10^7) seconds

Quick conversion:
requests/day ÷ 100,000 = requests/second (QPS)
```

---

## Data Size

```
1 byte     = 8 bits
1 KB       = 1,000 bytes (10^3)
1 MB       = 1,000,000 bytes (10^6)
1 GB       = 1,000,000,000 bytes (10^9)
1 TB       = 1,000,000,000,000 bytes (10^12)
1 PB       = 1,000,000,000,000,000 bytes (10^15)

Common sizes:
char       = 1-4 bytes (UTF-8)
integer    = 4 bytes
long       = 8 bytes
double     = 8 bytes
UUID       = 36 bytes (as string)
timestamp  = 8 bytes
```

---

## Common Data Sizes

| Data Type | Approximate Size |
|-----------|------------------|
| Tweet | 280 chars = ~300 bytes |
| URL | 100 chars = 100 bytes |
| Email address | 50 bytes |
| User profile (basic) | 1 KB |
| Image thumbnail | 10-50 KB |
| Web page | 2-5 MB |
| Image (high quality) | 200 KB - 2 MB |
| Video (1 min, compressed) | 5-10 MB |
| HD Video (1 min) | 100-150 MB |

---

## Latency Numbers

```
L1 cache reference:              0.5 ns
L2 cache reference:              7 ns
RAM reference:                   100 ns
SSD random read:                 150 μs (150,000 ns)
HDD random read:                 10 ms (10,000,000 ns)
Same datacenter round trip:      0.5 ms
Cross-continent round trip:      150 ms

Memory is ~1000x faster than SSD
SSD is ~100x faster than HDD
```

---

## Throughput

```
SSD sequential read:     500 MB/s
HDD sequential read:     100 MB/s
1 Gbps network:          125 MB/s
10 Gbps network:         1.25 GB/s

Redis:                   100K+ ops/sec
Memcached:               100K+ ops/sec
PostgreSQL:              10K-50K queries/sec
MySQL:                   10K-50K queries/sec
Kafka:                   millions of messages/sec
```

---

## Availability

```
Availability    Downtime/Year    Downtime/Month
99%             3.65 days        7.2 hours
99.9%           8.76 hours       43.8 minutes
99.95%          4.38 hours       21.9 minutes
99.99%          52.6 minutes     4.38 minutes
99.999%         5.26 minutes     26.3 seconds
```

---

## Quick Formulas

### QPS Calculation
```
DAU × actions_per_user / seconds_per_day = QPS
QPS × 2 or 3 = peak QPS
```

### Storage Calculation
```
records_per_day × record_size = daily_storage
daily_storage × 365 × years = total_storage
total_storage × 3 = storage_with_replication
```

### Bandwidth Calculation
```
QPS × response_size = outgoing_bandwidth
writes_per_second × request_size = incoming_bandwidth
```

### Server Calculation
```
peak_QPS / single_server_QPS = number_of_servers
(add 20-30% for headroom)
```

---

## Example: Social Media App

```
Given:
- 100M DAU
- Each user: 10 reads, 1 write per day
- Average post: 500 bytes
- Store for 5 years

Calculations:

Traffic:
- Reads: 100M × 10 = 1B/day = 10K QPS
- Writes: 100M × 1 = 100M/day = 1K QPS
- Peak: ~30K read QPS, ~3K write QPS

Storage:
- 100M posts/day × 500 bytes = 50 GB/day
- 50 GB × 365 × 5 = 91 TB
- With 3x replication = 273 TB

Bandwidth:
- Outgoing: 10K QPS × 500 bytes = 5 MB/s
- Peak: 15 MB/s
```

---

## Example: Chat Application

```
Given:
- 50M DAU
- Each user sends 50 messages/day
- Average message: 100 bytes

Calculations:

Traffic:
- Messages: 50M × 50 = 2.5B/day
- QPS: 2.5B / 100K = 25K messages/sec
- Peak: 75K messages/sec

Storage:
- 2.5B × 100 bytes = 250 GB/day
- Per year: 91 TB

Bandwidth:
- 25K × 100 bytes = 2.5 MB/s
- Peak: 7.5 MB/s
```

---

## Example: Video Streaming

```
Given:
- 10M concurrent viewers
- Video bitrate: 5 Mbps
- 50% from CDN cache

Calculations:

Bandwidth:
- Total: 10M × 5 Mbps = 50 Pbps
- From origin (50% miss): 25 Pbps
- This is why CDNs are essential!

Storage (1 day of uploads):
- 1M videos/day × 100 MB = 100 TB/day
- Need multiple quality levels: 100 TB × 5 = 500 TB/day
```

---

## Common Ratios

```
Read:Write ratios:
- Social media: 100:1
- E-commerce: 50:1
- Chat: 1:1
- Logging: 1:100

Cache hit ratio:
- Good: > 90%
- Great: > 95%

Replication:
- Standard: 3 replicas
- Critical: 5+ replicas
```
