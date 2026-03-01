# Benchmarking Databases

## Benchmarking Fundamentals

```
┌─────────────────────────────────────────────────────────────────┐
│              Why Benchmark?                                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  PURPOSES                                                       │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Database selection (compare options)                    │ │
│  │ • Capacity planning (how much can it handle?)             │ │
│  │ • Regression testing (did changes hurt performance?)      │ │
│  │ • Configuration validation (is tuning effective?)         │ │
│  │ • Hardware evaluation (which specs needed?)               │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  BENCHMARK TYPES                                                │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Synthetic: Standardized workloads (TPC-C, YCSB)           │ │
│  │ + Comparable across systems                                │ │
│  │ + Reproducible                                             │ │
│  │ - May not reflect your workload                           │ │
│  │                                                             │ │
│  │ Application-specific: Your actual queries                 │ │
│  │ + Reflects real performance                                │ │
│  │ + Identifies actual bottlenecks                           │ │
│  │ - More effort to set up                                   │ │
│  │ - Harder to compare across systems                        │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Standard Benchmarks

```
┌─────────────────────────────────────────────────────────────────┐
│              Industry Standard Benchmarks                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  TPC-C (OLTP)                                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Simulates wholesale supplier workload                   │ │
│  │ • Mix of 5 transaction types                              │ │
│  │ • Measures: transactions per minute (tpmC)                │ │
│  │ • Standard for OLTP comparison                            │ │
│  │                                                             │ │
│  │ Transaction mix:                                           │ │
│  │ - New Order (45%)                                          │ │
│  │ - Payment (43%)                                            │ │
│  │ - Order Status (4%)                                        │ │
│  │ - Delivery (4%)                                            │ │
│  │ - Stock Level (4%)                                         │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  TPC-H (OLAP)                                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Decision support benchmark                              │ │
│  │ • 22 complex analytical queries                           │ │
│  │ • Tests: joins, aggregations, subqueries                  │ │
│  │ • Measures: queries per hour (QphH)                       │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  YCSB (Yahoo Cloud Serving Benchmark)                           │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • For key-value and NoSQL stores                          │ │
│  │ • Configurable workload patterns                          │ │
│  │ • Standard workloads:                                     │ │
│  │   - A: Update heavy (50/50 read/write)                   │ │
│  │   - B: Read heavy (95/5 read/write)                      │ │
│  │   - C: Read only (100% read)                             │ │
│  │   - D: Read latest (95% read, insert new)                │ │
│  │   - E: Short ranges (95% scan, 5% insert)                │ │
│  │   - F: Read-modify-write                                 │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  PGBENCH (PostgreSQL)                                           │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Built-in PostgreSQL benchmark                           │ │
│  │ • Based on TPC-B (simple banking transactions)            │ │
│  │ • Easy to run, good for quick comparisons                 │ │
│  │                                                             │ │
│  │ $ pgbench -i -s 100 mydb        # Initialize              │ │
│  │ $ pgbench -c 10 -j 2 -T 60 mydb # Run 60s, 10 clients    │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  SYSBENCH                                                       │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Multi-purpose benchmark tool                            │ │
│  │ • Supports MySQL, PostgreSQL                              │ │
│  │ • OLTP read/write workloads                               │ │
│  │                                                             │ │
│  │ $ sysbench oltp_read_write \                              │ │
│  │     --mysql-host=localhost \                              │ │
│  │     --tables=10 --table-size=1000000 \                    │ │
│  │     --threads=16 --time=300 run                           │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Benchmarking Methodology

```
┌─────────────────────────────────────────────────────────────────┐
│              Benchmarking Best Practices                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  PREPARATION                                                    │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ 1. Define clear goals and metrics                         │ │
│  │ 2. Use production-representative data sizes               │ │
│  │ 3. Document system configuration                          │ │
│  │ 4. Isolate the system (no other workloads)                │ │
│  │ 5. Warm up caches before measuring                        │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  EXECUTION                                                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ 1. Run multiple iterations (minimum 3)                    │ │
│  │ 2. Vary parameters systematically                         │ │
│  │ 3. Monitor resource utilization                           │ │
│  │ 4. Record all measurements                                │ │
│  │ 5. Watch for steady state                                 │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ANALYSIS                                                       │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ 1. Report percentiles, not just averages                  │ │
│  │ 2. Include standard deviation                             │ │
│  │ 3. Identify bottlenecks (CPU, I/O, network?)              │ │
│  │ 4. Compare apples to apples                               │ │
│  │ 5. Document methodology for reproducibility               │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  COMMON PITFALLS                                                │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ ✗ Cold cache vs warm cache comparisons                   │ │
│  │ ✗ Ignoring variance in results                           │ │
│  │ ✗ Not testing at production scale                        │ │
│  │ ✗ Benchmark client becoming bottleneck                   │ │
│  │ ✗ Not accounting for network latency                     │ │
│  │ ✗ Testing on different hardware configs                  │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Custom Benchmark Design

```
┌─────────────────────────────────────────────────────────────────┐
│              Designing Application-Specific Benchmarks           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  STEP 1: ANALYZE PRODUCTION WORKLOAD                            │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Collect:                                                   │ │
│  │ • Top queries by frequency                                 │ │
│  │ • Top queries by execution time                            │ │
│  │ • Read/write ratio                                         │ │
│  │ • Peak vs average load                                     │ │
│  │ • Data access patterns                                     │ │
│  │                                                             │ │
│  │ Tools:                                                      │ │
│  │ • pg_stat_statements (PostgreSQL)                          │ │
│  │ • Performance Schema (MySQL)                               │ │
│  │ • Query logs analysis                                      │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  STEP 2: CREATE REPRESENTATIVE WORKLOAD                         │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Example workload definition:                               │ │
│  │                                                             │ │
│  │ Query Type          Frequency    Target Latency           │ │
│  │ ─────────────────────────────────────────────────────────  │ │
│  │ Get user by ID      40%          < 5ms                    │ │
│  │ Search products     25%          < 50ms                   │ │
│  │ List orders         20%          < 20ms                   │ │
│  │ Create order        10%          < 100ms                  │ │
│  │ Update inventory    5%           < 50ms                   │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  STEP 3: GENERATE REALISTIC DATA                                │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Match production data distribution                       │ │
│  │ • Include data skew (Zipf distribution)                   │ │
│  │ • Scale appropriately (10%, 50%, 100%)                    │ │
│  │ • Anonymize sensitive data                                 │ │
│  │                                                             │ │
│  │ Tools: Faker, data generators, production snapshots        │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  STEP 4: BUILD BENCHMARK FRAMEWORK                              │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ # Example: Python benchmark script                         │ │
│  │ import random                                               │ │
│  │ import time                                                 │ │
│  │ from statistics import mean, stdev                         │ │
│  │                                                             │ │
│  │ def benchmark(connection, iterations=1000):                │ │
│  │     latencies = []                                         │ │
│  │     for _ in range(iterations):                            │ │
│  │         query = select_random_query()                      │ │
│  │         start = time.perf_counter()                        │ │
│  │         execute(connection, query)                         │ │
│  │         latencies.append(time.perf_counter() - start)      │ │
│  │                                                             │ │
│  │     return {                                                │ │
│  │         'p50': percentile(latencies, 50),                  │ │
│  │         'p95': percentile(latencies, 95),                  │ │
│  │         'p99': percentile(latencies, 99),                  │ │
│  │         'qps': iterations / sum(latencies)                 │ │
│  │     }                                                       │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Interpreting Results

```
┌─────────────────────────────────────────────────────────────────┐
│              Understanding Benchmark Results                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  LATENCY DISTRIBUTION                                           │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  Requests │                                                 │ │
│  │     ▲     │                                                 │ │
│  │     │  ▓▓▓▓                                                │ │
│  │     │  ▓▓▓▓▓▓                                              │ │
│  │     │  ▓▓▓▓▓▓▓▓                                            │ │
│  │     │  ▓▓▓▓▓▓▓▓▓▓▓▓░░░░░░░░░▒                             │ │
│  │     └──────────────────────────────────▶ Latency           │ │
│  │        ▲      ▲         ▲        ▲                         │ │
│  │        │      │         │        │                          │ │
│  │       p50    p95       p99      max                        │ │
│  │                                                             │ │
│  │  Tail latency (p99, p999) matters for user experience      │ │
│  │  Average can hide problems in the tail                     │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  THROUGHPUT VS LATENCY CURVE                                    │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  Latency│                                     ╱            │ │
│  │    ▲    │                                   ╱              │ │
│  │    │    │                                 ╱                │ │
│  │    │    │                     ┌─────────╱ ← Saturation     │ │
│  │    │    │                    ╱                              │ │
│  │    │    │          ─────────╱                               │ │
│  │    │    │─────────╱                                         │ │
│  │    └────┴───────────────────────────────────▶ Load (QPS)   │ │
│  │              ▲                        ▲                     │ │
│  │              │                        │                     │ │
│  │         Optimal                  Max throughput             │ │
│  │         operating               (latency explodes)          │ │
│  │         point                                               │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  Operating beyond the "knee" causes latency spikes             │
└─────────────────────────────────────────────────────────────────┘
```
