# Load Testing

## Load Testing Fundamentals

```
┌─────────────────────────────────────────────────────────────────┐
│              Load Testing Types                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  LOAD TEST                                                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Purpose: Verify system handles expected load               │ │
│  │                                                             │ │
│  │ Load                                                        │ │
│  │  ▲      ┌───────────────────────────┐                      │ │
│  │  │      │   Expected Load Level     │                      │ │
│  │  │  ────┘                           └────                  │ │
│  │  └───────────────────────────────────────▶ Time            │ │
│  │                                                             │ │
│  │ Validates: Normal operation performance                    │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  STRESS TEST                                                    │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Purpose: Find breaking point                               │ │
│  │                                                             │ │
│  │ Load                                                        │ │
│  │  ▲                        ╱╲ ← Breaking point              │ │
│  │  │                      ╱    ╲                             │ │
│  │  │                    ╱        ╲                           │ │
│  │  │               ╱───           ───                        │ │
│  │  │         ╱────                                            │ │
│  │  │  ──────╱                                                 │ │
│  │  └───────────────────────────────────────▶ Time            │ │
│  │                                                             │ │
│  │ Identifies: Maximum capacity, failure modes                │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  SPIKE TEST                                                     │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Purpose: Test sudden traffic bursts                        │ │
│  │                                                             │ │
│  │ Load                                                        │ │
│  │  ▲       │         │                                       │ │
│  │  │      ┌┴┐       ┌┴┐                                      │ │
│  │  │      │ │       │ │                                      │ │
│  │  │  ────┘ └───────┘ └────                                  │ │
│  │  └───────────────────────────────────────▶ Time            │ │
│  │                                                             │ │
│  │ Validates: Burst handling, auto-scaling                    │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  SOAK TEST (Endurance)                                          │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Purpose: Find issues over extended time                    │ │
│  │                                                             │ │
│  │ Load                                                        │ │
│  │  ▲      ┌─────────────────────────────────────────┐        │ │
│  │  │      │   Sustained Load (hours/days)           │        │ │
│  │  │  ────┘                                         └──      │ │
│  │  └───────────────────────────────────────▶ Time            │ │
│  │                                                             │ │
│  │ Identifies: Memory leaks, resource exhaustion              │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Load Testing Tools

```
┌─────────────────────────────────────────────────────────────────┐
│              Load Testing Tools                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  K6 (Modern, Developer-friendly)                                │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ // k6 script                                                │ │
│  │ import http from 'k6/http';                                │ │
│  │ import sql from 'k6/x/sql';                                │ │
│  │                                                             │ │
│  │ export const options = {                                    │ │
│  │   stages: [                                                 │ │
│  │     { duration: '2m', target: 100 }, // Ramp up            │ │
│  │     { duration: '5m', target: 100 }, // Steady             │ │
│  │     { duration: '2m', target: 0 },   // Ramp down          │ │
│  │   ],                                                        │ │
│  │ };                                                          │ │
│  │                                                             │ │
│  │ export default function() {                                 │ │
│  │   const res = http.get('http://api.example.com/users');    │ │
│  │   check(res, { 'status is 200': (r) => r.status === 200 });│ │
│  │ }                                                           │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  LOCUST (Python-based)                                          │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ from locust import HttpUser, task, between                 │ │
│  │                                                             │ │
│  │ class WebUser(HttpUser):                                    │ │
│  │     wait_time = between(1, 3)                              │ │
│  │                                                             │ │
│  │     @task(3)                                                │ │
│  │     def get_products(self):                                 │ │
│  │         self.client.get("/api/products")                   │ │
│  │                                                             │ │
│  │     @task(1)                                                │ │
│  │     def create_order(self):                                 │ │
│  │         self.client.post("/api/orders", json={...})        │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  PGBENCH (PostgreSQL-specific)                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ # Custom script                                             │ │
│  │ $ cat custom.sql                                            │ │
│  │ \set user_id random(1, 1000000)                            │ │
│  │ SELECT * FROM users WHERE id = :user_id;                   │ │
│  │                                                             │ │
│  │ # Run with custom script                                   │ │
│  │ $ pgbench -c 50 -j 10 -T 300 -f custom.sql mydb            │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  JMETER (Enterprise-grade)                                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • GUI-based test plan creation                             │ │
│  │ • JDBC sampler for direct DB testing                       │ │
│  │ • Distributed load generation                              │ │
│  │ • Extensive reporting                                       │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Database-Specific Load Testing

```
┌─────────────────────────────────────────────────────────────────┐
│              Direct Database Load Testing                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  SYSBENCH (MySQL/PostgreSQL)                                    │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ # Prepare test data                                        │ │
│  │ sysbench oltp_read_write \                                 │ │
│  │   --mysql-host=localhost \                                 │ │
│  │   --mysql-user=test \                                      │ │
│  │   --mysql-password=test \                                  │ │
│  │   --mysql-db=sbtest \                                      │ │
│  │   --tables=10 \                                            │ │
│  │   --table-size=1000000 \                                   │ │
│  │   prepare                                                   │ │
│  │                                                             │ │
│  │ # Run test                                                  │ │
│  │ sysbench oltp_read_write \                                 │ │
│  │   --threads=64 \                                           │ │
│  │   --time=600 \                                             │ │
│  │   --report-interval=10 \                                   │ │
│  │   run                                                       │ │
│  │                                                             │ │
│  │ # Workload types:                                          │ │
│  │ # oltp_read_only, oltp_write_only, oltp_read_write        │ │
│  │ # oltp_point_select, oltp_update_index                    │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  HAMMERDB (TPC-C/TPC-H)                                         │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Full TPC-C and TPC-H implementation                      │ │
│  │ • Supports Oracle, SQL Server, MySQL, PostgreSQL          │ │
│  │ • GUI and CLI interfaces                                   │ │
│  │ • Industry-standard benchmarks                             │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  YCSB (NoSQL)                                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ # Load data                                                 │ │
│  │ ./bin/ycsb load mongodb -s \                               │ │
│  │   -P workloads/workloada \                                 │ │
│  │   -p mongodb.url=mongodb://localhost:27017                 │ │
│  │                                                             │ │
│  │ # Run workload                                              │ │
│  │ ./bin/ycsb run mongodb -s \                                │ │
│  │   -P workloads/workloada \                                 │ │
│  │   -threads 32 \                                            │ │
│  │   -target 10000                                            │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Load Test Design

```
┌─────────────────────────────────────────────────────────────────┐
│              Load Test Scenario Design                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  REALISTIC WORKLOAD MODELING                                    │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ 1. Analyze production traffic patterns                     │ │
│  │    • Peak hours                                            │ │
│  │    • Seasonal variations                                   │ │
│  │    • Geographic distribution                               │ │
│  │                                                             │ │
│  │ 2. Model user behavior                                      │ │
│  │    • Think time between actions                            │ │
│  │    • Session duration                                      │ │
│  │    • Action sequences                                      │ │
│  │                                                             │ │
│  │ 3. Include data variety                                    │ │
│  │    • Different query types                                 │ │
│  │    • Various data sizes                                    │ │
│  │    • Hot vs cold data access                               │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  LOAD PROFILE EXAMPLE                                           │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Virtual Users (VUs) over time:                             │ │
│  │                                                             │ │
│  │ Phase 1: Ramp Up (10 min)                                  │ │
│  │   └─ VUs: 0 → 500                                          │ │
│  │                                                             │ │
│  │ Phase 2: Steady State (30 min)                             │ │
│  │   └─ VUs: 500 (constant)                                   │ │
│  │                                                             │ │
│  │ Phase 3: Peak (10 min)                                     │ │
│  │   └─ VUs: 500 → 1000                                       │ │
│  │                                                             │ │
│  │ Phase 4: Sustained Peak (15 min)                           │ │
│  │   └─ VUs: 1000 (constant)                                  │ │
│  │                                                             │ │
│  │ Phase 5: Ramp Down (5 min)                                 │ │
│  │   └─ VUs: 1000 → 0                                         │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  PASS/FAIL CRITERIA                                             │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Response Time:                                              │ │
│  │ • p50 < 100ms                                              │ │
│  │ • p95 < 500ms                                              │ │
│  │ • p99 < 1000ms                                             │ │
│  │                                                             │ │
│  │ Error Rate:                                                 │ │
│  │ • < 0.1% errors                                            │ │
│  │                                                             │ │
│  │ Throughput:                                                 │ │
│  │ • > 1000 requests/second sustained                         │ │
│  │                                                             │ │
│  │ Resource Usage:                                             │ │
│  │ • CPU < 80%                                                 │ │
│  │ • Memory < 85%                                              │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Analyzing Load Test Results

```
┌─────────────────────────────────────────────────────────────────┐
│              Result Analysis                                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  KEY OBSERVATIONS                                               │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Look for:                                                   │ │
│  │                                                             │ │
│  │ • Latency degradation under load                           │ │
│  │   ┌──────────────────────────────────────────────────┐    │ │
│  │   │ At 100 VUs: p99 = 50ms                           │    │ │
│  │   │ At 500 VUs: p99 = 200ms  ← Degradation starts   │    │ │
│  │   │ At 1000 VUs: p99 = 2000ms ← Severe              │    │ │
│  │   └──────────────────────────────────────────────────┘    │ │
│  │                                                             │ │
│  │ • Error rate increases                                     │ │
│  │   ┌──────────────────────────────────────────────────┐    │ │
│  │   │ At 500 VUs: 0.01% errors                         │    │ │
│  │   │ At 800 VUs: 0.1% errors ← Connection timeouts   │    │ │
│  │   │ At 1000 VUs: 5% errors ← System overloaded      │    │ │
│  │   └──────────────────────────────────────────────────┘    │ │
│  │                                                             │ │
│  │ • Resource saturation                                      │ │
│  │   ┌──────────────────────────────────────────────────┐    │ │
│  │   │ CPU: 95% ← Bottleneck identified               │    │ │
│  │   │ Memory: 70%                                      │    │ │
│  │   │ Disk I/O: 30%                                    │    │ │
│  │   │ Connections: 480/500 ← Near limit              │    │ │
│  │   └──────────────────────────────────────────────────┘    │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  BOTTLENECK IDENTIFICATION                                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ If CPU saturated → Query optimization, add replicas       │ │
│  │ If Memory saturated → Increase RAM, optimize queries      │ │
│  │ If Disk I/O saturated → Faster storage, better indexing   │ │
│  │ If Connections exhausted → Connection pooling             │ │
│  │ If Network saturated → Reduce payload, caching            │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```
