# Capacity Planning

## Capacity Planning Fundamentals

```
┌─────────────────────────────────────────────────────────────────┐
│              Capacity Planning Overview                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  GOAL: Ensure system can handle future workload demands         │
│                                                                  │
│  KEY QUESTIONS                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ 1. What is current capacity utilization?                  │ │
│  │ 2. How is demand growing?                                  │ │
│  │ 3. When will we hit limits?                               │ │
│  │ 4. What resources need to scale?                          │ │
│  │ 5. What are the cost implications?                        │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  CAPACITY PLANNING PROCESS                                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐ │ │
│  │  │ Measure │───▶│ Model   │───▶│ Forecast│───▶│  Plan   │ │ │
│  │  │ Current │    │ Capacity│    │ Demand  │    │ Actions │ │ │
│  │  └─────────┘    └─────────┘    └─────────┘    └─────────┘ │ │
│  │       ▲                                             │      │ │
│  │       └─────────────────────────────────────────────┘      │ │
│  │                    Continuous cycle                        │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Measuring Current Capacity

```
┌─────────────────────────────────────────────────────────────────┐
│              Current State Assessment                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  RESOURCE METRICS TO COLLECT                                    │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ COMPUTE                                                    │ │
│  │ • CPU utilization (average, peak)                          │ │
│  │ • Memory usage                                             │ │
│  │ • Connection count                                          │ │
│  │                                                             │ │
│  │ STORAGE                                                     │ │
│  │ • Disk space used/available                                │ │
│  │ • IOPS (reads/writes per second)                           │ │
│  │ • Throughput (MB/s)                                        │ │
│  │ • Latency (read/write)                                     │ │
│  │                                                             │ │
│  │ NETWORK                                                     │ │
│  │ • Bandwidth utilization                                    │ │
│  │ • Packets per second                                       │ │
│  │ • Replication traffic                                      │ │
│  │                                                             │ │
│  │ DATABASE                                                    │ │
│  │ • Queries per second                                       │ │
│  │ • Active connections                                       │ │
│  │ • Buffer pool hit ratio                                    │ │
│  │ • Replication lag                                          │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  UTILIZATION THRESHOLDS                                         │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Resource    │ Safe   │ Warning │ Critical                 │ │
│  │ ────────────┼────────┼─────────┼──────────                 │ │
│  │ CPU         │ < 60%  │ 60-80%  │ > 80%                     │ │
│  │ Memory      │ < 70%  │ 70-85%  │ > 85%                     │ │
│  │ Disk Space  │ < 60%  │ 60-80%  │ > 80%                     │ │
│  │ Disk I/O    │ < 50%  │ 50-75%  │ > 75%                     │ │
│  │ Connections │ < 60%  │ 60-80%  │ > 80%                     │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  Keep headroom for spikes and growth                            │
└─────────────────────────────────────────────────────────────────┘
```

## Demand Forecasting

```
┌─────────────────────────────────────────────────────────────────┐
│              Forecasting Future Demand                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  HISTORICAL TREND ANALYSIS                                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  QPS  │                                          ╱╱        │ │
│  │   ▲   │                                      ╱╱            │ │
│  │   │   │                                  ╱╱                │ │
│  │   │   │                        ─────╱╱╱╱   ← Projected    │ │
│  │   │   │           ───────────╱                              │ │
│  │   │   │  ─────────                                          │ │
│  │   │   │╱                      ← Historical                  │ │
│  │   └───┴─────────────────────────────────────▶ Time         │ │
│  │       Jan   Apr   Jul   Oct   Jan   Apr                    │ │
│  │                                                             │ │
│  │  Linear regression on historical data                      │ │
│  │  Account for seasonality (holidays, events)                │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  GROWTH FACTORS TO CONSIDER                                     │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Organic growth:                                             │ │
│  │ • User growth rate                                         │ │
│  │ • Feature adoption                                         │ │
│  │ • Data accumulation rate                                   │ │
│  │                                                             │ │
│  │ Step changes:                                               │ │
│  │ • New product launches                                     │ │
│  │ • Marketing campaigns                                      │ │
│  │ • Enterprise client onboarding                             │ │
│  │ • Geographic expansion                                     │ │
│  │                                                             │ │
│  │ Seasonal patterns:                                          │ │
│  │ • Holiday traffic spikes                                   │ │
│  │ • End of month/quarter processing                          │ │
│  │ • Time-of-day patterns                                     │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  FORECASTING MODELS                                             │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Simple: Linear extrapolation                               │ │
│  │   future = current × (1 + growth_rate) ^ months            │ │
│  │                                                             │ │
│  │ Moderate: Exponential smoothing                            │ │
│  │   Accounts for trend and seasonality                       │ │
│  │                                                             │ │
│  │ Complex: Time series analysis (ARIMA, Prophet)             │ │
│  │   ML-based, handles complex patterns                       │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Capacity Modeling

```
┌─────────────────────────────────────────────────────────────────┐
│              Building Capacity Models                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  RESOURCE-TO-WORKLOAD MAPPING                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Determine: How much resource per unit of work?             │ │
│  │                                                             │ │
│  │ Example measurements:                                       │ │
│  │ • 1 user session = 0.01 CPU cores                          │ │
│  │ • 1 user session = 50 MB memory                            │ │
│  │ • 1 active user = 0.5 connections                          │ │
│  │ • 1 transaction = 100 IOPS                                 │ │
│  │                                                             │ │
│  │ Capacity = Resources / Per-unit-consumption                │ │
│  │ Example: 32 cores / 0.01 = 3,200 concurrent users         │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  DATABASE-SPECIFIC CAPACITY FACTORS                             │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ STORAGE GROWTH                                              │ │
│  │ Current size: 500 GB                                       │ │
│  │ Monthly growth: 20 GB                                      │ │
│  │ Available: 1.5 TB                                          │ │
│  │ Runway: (1500 - 500) / 20 = 50 months                     │ │
│  │                                                             │ │
│  │ CONNECTION CAPACITY                                        │ │
│  │ Max connections: 500                                       │ │
│  │ Peak usage: 350                                            │ │
│  │ Growth rate: 10% monthly                                   │ │
│  │ Months until limit: log(500/350) / log(1.1) ≈ 4 months   │ │
│  │                                                             │ │
│  │ QUERY CAPACITY                                              │ │
│  │ Current QPS: 5,000                                         │ │
│  │ CPU headroom: 30%                                          │ │
│  │ Max sustainable QPS: 5000 / 0.70 ≈ 7,100                  │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  BOTTLENECK IDENTIFICATION                                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Resource       Current  Capacity  Utilization  Runway     │ │
│  │ ─────────────────────────────────────────────────────────  │ │
│  │ CPU            24 cores 32 cores  75%          3 months   │ │
│  │ Memory         48 GB    64 GB     75%          4 months   │ │
│  │ Storage        800 GB   1 TB      80%          5 months   │ │
│  │ Connections    400      500       80%          2 months ← │ │
│  │ IOPS           8000     10000     80%          3 months   │ │
│  │                                                             │ │
│  │ Connections will be the first bottleneck                  │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Scaling Strategies

```
┌─────────────────────────────────────────────────────────────────┐
│              Scaling Options                                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  VERTICAL SCALING (Scale Up)                                    │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Add more resources to existing instance                    │ │
│  │                                                             │ │
│  │ ┌─────────┐         ┌─────────────┐                        │ │
│  │ │ 4 cores │   →     │  16 cores   │                        │ │
│  │ │ 16 GB   │         │  64 GB      │                        │ │
│  │ │ 500 GB  │         │  2 TB NVMe  │                        │ │
│  │ └─────────┘         └─────────────┘                        │ │
│  │                                                             │ │
│  │ Pros: Simple, no code changes                              │ │
│  │ Cons: Limits exist, expensive at top tier, single point   │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  HORIZONTAL SCALING (Scale Out)                                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Add more instances                                          │ │
│  │                                                             │ │
│  │ ┌─────────┐       ┌─────────┐  ┌─────────┐  ┌─────────┐   │ │
│  │ │ Primary │   →   │ Primary │  │ Replica │  │ Replica │   │ │
│  │ └─────────┘       └─────────┘  └─────────┘  └─────────┘   │ │
│  │                                                             │ │
│  │ Read scaling: Add read replicas                            │ │
│  │ Write scaling: Sharding (more complex)                     │ │
│  │                                                             │ │
│  │ Pros: Near-linear scaling, high availability               │ │
│  │ Cons: Complexity, potential consistency issues             │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  OPTIMIZATION (Do More with Less)                               │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Before scaling hardware, consider:                         │ │
│  │                                                             │ │
│  │ • Query optimization                                       │ │
│  │ • Better indexing                                          │ │
│  │ • Caching layer                                            │ │
│  │ • Connection pooling                                       │ │
│  │ • Data archiving                                           │ │
│  │ • Schema denormalization                                   │ │
│  │                                                             │ │
│  │ Often cheaper and faster than adding resources             │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Capacity Planning Checklist

```
┌─────────────────────────────────────────────────────────────────┐
│              Capacity Planning Checklist                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  REGULAR REVIEWS (Monthly)                                      │
│  □ Review current utilization metrics                          │
│  □ Update growth projections                                    │
│  □ Check runway for each resource                               │
│  □ Identify upcoming capacity events                            │
│                                                                  │
│  BEFORE LAUNCHES                                                │
│  □ Estimate additional load                                     │
│  □ Load test with projected traffic                             │
│  □ Provision additional capacity if needed                      │
│  □ Have rollback plan                                           │
│                                                                  │
│  QUARTERLY PLANNING                                             │
│  □ Review 6-month capacity forecast                             │
│  □ Budget for infrastructure growth                             │
│  □ Plan major scaling initiatives                               │
│  □ Review architectural improvements                            │
│                                                                  │
│  DOCUMENTATION                                                  │
│  □ Current architecture diagram                                 │
│  □ Resource inventory                                           │
│  □ Capacity thresholds and alerts                               │
│  □ Scaling runbooks                                             │
│  □ Cost projections                                             │
└─────────────────────────────────────────────────────────────────┘
```
