# Monitoring and Alerting

## Monitoring Strategy

```
┌─────────────────────────────────────────────────────────────────┐
│              Database Monitoring Layers                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Layer 4: BUSINESS METRICS                               │    │
│  │  Orders/second, Revenue, User signups                    │    │
│  └─────────────────────────────────────────────────────────┘    │
│                           ▲                                      │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Layer 3: APPLICATION METRICS                            │    │
│  │  API latency, Error rates, Request throughput            │    │
│  └─────────────────────────────────────────────────────────┘    │
│                           ▲                                      │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Layer 2: DATABASE METRICS                               │    │
│  │  QPS, Connections, Replication lag, Cache hit ratio      │    │
│  └─────────────────────────────────────────────────────────┘    │
│                           ▲                                      │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  Layer 1: INFRASTRUCTURE METRICS                         │    │
│  │  CPU, Memory, Disk I/O, Network                          │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                  │
│  Monitor all layers for complete visibility                     │
└─────────────────────────────────────────────────────────────────┘
```

## Key Metrics to Monitor

```
┌─────────────────────────────────────────────────────────────────┐
│              Essential Database Metrics                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  PERFORMANCE                                                    │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Query latency (p50, p95, p99)                              │ │
│  │ ├─ By query type (SELECT, INSERT, UPDATE)                 │ │
│  │ └─ By table/endpoint                                       │ │
│  │                                                             │ │
│  │ Throughput                                                  │ │
│  │ ├─ Queries per second                                      │ │
│  │ ├─ Transactions per second                                 │ │
│  │ └─ Rows read/written per second                            │ │
│  │                                                             │ │
│  │ Slow queries                                                │ │
│  │ ├─ Count of queries > threshold                            │ │
│  │ └─ Percentage of slow queries                              │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  RESOURCE UTILIZATION                                           │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ CPU usage (user, system, iowait)                           │ │
│  │ Memory (used, cached, buffer pool)                         │ │
│  │ Disk (space, IOPS, throughput, latency)                    │ │
│  │ Network (bytes in/out, packets, errors)                    │ │
│  │ Connections (active, idle, waiting)                        │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  DATABASE-SPECIFIC                                              │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Buffer pool / cache hit ratio                              │ │
│  │ Lock waits and deadlocks                                   │ │
│  │ Replication lag                                            │ │
│  │ Checkpoint frequency and duration                          │ │
│  │ WAL/binlog size and growth                                 │ │
│  │ Table/index bloat                                          │ │
│  │ Vacuum/analyze activity (PostgreSQL)                       │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  AVAILABILITY                                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Uptime / availability percentage                           │ │
│  │ Connection success rate                                    │ │
│  │ Failover events                                            │ │
│  │ Replica health                                             │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Monitoring Tools

```
┌─────────────────────────────────────────────────────────────────┐
│              Monitoring Tool Stack                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  METRICS COLLECTION                                             │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Prometheus                                                  │ │
│  │ • Pull-based metrics collection                            │ │
│  │ • Powerful query language (PromQL)                         │ │
│  │ • Built-in alerting                                        │ │
│  │                                                             │ │
│  │ Exporters:                                                  │ │
│  │ • postgres_exporter (PostgreSQL)                           │ │
│  │ • mysqld_exporter (MySQL)                                  │ │
│  │ • mongodb_exporter (MongoDB)                               │ │
│  │ • redis_exporter (Redis)                                   │ │
│  │ • node_exporter (system metrics)                           │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  VISUALIZATION                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Grafana                                                     │ │
│  │ • Rich dashboards                                          │ │
│  │ • Multiple data sources                                    │ │
│  │ • Pre-built database dashboards available                  │ │
│  │                                                             │ │
│  │ Example dashboard panels:                                   │ │
│  │ • QPS over time                                            │ │
│  │ • Latency percentiles                                      │ │
│  │ • Connection pool status                                   │ │
│  │ • Replication lag                                          │ │
│  │ • Resource utilization gauges                              │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  DATABASE-NATIVE TOOLS                                          │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ PostgreSQL:                                                 │ │
│  │ • pg_stat_statements (query statistics)                    │ │
│  │ • pg_stat_activity (active connections)                    │ │
│  │ • pgBadger (log analysis)                                  │ │
│  │                                                             │ │
│  │ MySQL:                                                      │ │
│  │ • Performance Schema                                        │ │
│  │ • sys schema                                                │ │
│  │ • Slow query log                                           │ │
│  │ • MySQL Enterprise Monitor                                 │ │
│  │                                                             │ │
│  │ MongoDB:                                                    │ │
│  │ • mongostat, mongotop                                      │ │
│  │ • Database Profiler                                        │ │
│  │ • MongoDB Atlas monitoring                                 │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  COMMERCIAL/MANAGED OPTIONS                                     │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Datadog (APM + infrastructure)                           │ │
│  │ • New Relic                                                 │ │
│  │ • Cloud-native: AWS CloudWatch, GCP Cloud Monitoring       │ │
│  │ • Percona Monitoring and Management (PMM)                  │ │
│  │ • pganalyze (PostgreSQL-specific)                          │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Alerting Strategy

```
┌─────────────────────────────────────────────────────────────────┐
│              Effective Alerting                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ALERT SEVERITY LEVELS                                          │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ CRITICAL (Page immediately)                                │ │
│  │ • Database down                                            │ │
│  │ • Replication completely broken                            │ │
│  │ • Disk > 95% full                                          │ │
│  │ • Error rate > 5%                                          │ │
│  │                                                             │ │
│  │ WARNING (Page during business hours)                       │ │
│  │ • High CPU > 80% for 10 min                                │ │
│  │ • Disk > 80% full                                          │ │
│  │ • Replication lag > 30 seconds                             │ │
│  │ • Connection pool > 80%                                    │ │
│  │                                                             │ │
│  │ INFO (Ticket/Dashboard only)                               │ │
│  │ • Slow query count increased                               │ │
│  │ • Cache hit ratio degraded                                 │ │
│  │ • Approaching capacity thresholds                          │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ALERT DESIGN PRINCIPLES                                        │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ 1. Alert on symptoms, not causes                           │ │
│  │    ✓ "Error rate increased"                               │ │
│  │    ✗ "CPU is high"                                        │ │
│  │                                                             │ │
│  │ 2. Every alert should be actionable                        │ │
│  │    If you can't do anything, it's noise                   │ │
│  │                                                             │ │
│  │ 3. Include context in alert                                │ │
│  │    • Current value vs threshold                            │ │
│  │    • Link to runbook                                       │ │
│  │    • Link to relevant dashboard                            │ │
│  │                                                             │ │
│  │ 4. Avoid alert fatigue                                     │ │
│  │    • Tune thresholds regularly                             │ │
│  │    • Use hysteresis (different up/down thresholds)        │ │
│  │    • Group related alerts                                  │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  EXAMPLE ALERTS (Prometheus/Alertmanager)                       │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ - alert: DatabaseDown                                      │ │
│  │   expr: pg_up == 0                                         │ │
│  │   for: 1m                                                   │ │
│  │   labels:                                                   │ │
│  │     severity: critical                                     │ │
│  │   annotations:                                              │ │
│  │     summary: "PostgreSQL is down"                          │ │
│  │     runbook: "https://wiki/runbooks/pg-down"              │ │
│  │                                                             │ │
│  │ - alert: HighReplicationLag                                │ │
│  │   expr: pg_replication_lag_seconds > 30                    │ │
│  │   for: 5m                                                   │ │
│  │   labels:                                                   │ │
│  │     severity: warning                                      │ │
│  │   annotations:                                              │ │
│  │     summary: "Replication lag is {{ $value }}s"           │ │
│  │                                                             │ │
│  │ - alert: DiskSpaceLow                                      │ │
│  │   expr: (pg_database_size_bytes / node_disk_total) > 0.8  │ │
│  │   for: 10m                                                  │ │
│  │   labels:                                                   │ │
│  │     severity: warning                                      │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Dashboards and Visualization

```
┌─────────────────────────────────────────────────────────────────┐
│              Dashboard Best Practices                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  DASHBOARD HIERARCHY                                            │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  ┌─────────────────────────────────────────────────────┐   │ │
│  │  │             Overview Dashboard                       │   │ │
│  │  │  High-level health, key metrics, SLIs               │   │ │
│  │  └────────────────────────┬────────────────────────────┘   │ │
│  │                           │                                 │ │
│  │         ┌─────────────────┼─────────────────┐              │ │
│  │         │                 │                 │               │ │
│  │         ▼                 ▼                 ▼               │ │
│  │  ┌───────────┐     ┌───────────┐     ┌───────────┐        │ │
│  │  │ Service A │     │ Service B │     │ Database  │        │ │
│  │  │ Dashboard │     │ Dashboard │     │ Dashboard │        │ │
│  │  └───────────┘     └───────────┘     └─────┬─────┘        │ │
│  │                                             │               │ │
│  │                              ┌──────────────┼──────────────┐│ │
│  │                              │              │              ││ │
│  │                              ▼              ▼              ▼│ │
│  │                        ┌─────────┐    ┌─────────┐    ┌────────┐
│  │                        │ Queries │    │ Storage │    │ Replica│
│  │                        │  Deep   │    │  Deep   │    │  Deep  │
│  │                        │  Dive   │    │  Dive   │    │  Dive  │
│  │                        └─────────┘    └─────────┘    └────────┘
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  DATABASE DASHBOARD SECTIONS                                    │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ 1. Health Status (up/down, replication status)            │ │
│  │ 2. Query Performance (QPS, latency, slow queries)         │ │
│  │ 3. Resource Utilization (CPU, memory, disk, connections)  │ │
│  │ 4. Replication (lag, throughput, errors)                  │ │
│  │ 5. Cache Performance (hit ratio, evictions)               │ │
│  │ 6. Storage (size, growth rate, IOPS)                      │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Incident Response

```
┌─────────────────────────────────────────────────────────────────┐
│              Database Incident Response                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  INCIDENT WORKFLOW                                              │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ 1. DETECT                                                   │ │
│  │    • Alert fires or user reports issue                     │ │
│  │    • Acknowledge alert                                      │ │
│  │                                                             │ │
│  │ 2. ASSESS                                                   │ │
│  │    • Check dashboards for scope                            │ │
│  │    • Identify affected services                            │ │
│  │    • Determine severity                                    │ │
│  │                                                             │ │
│  │ 3. COMMUNICATE                                              │ │
│  │    • Update status page                                    │ │
│  │    • Notify stakeholders                                   │ │
│  │    • Start incident channel                                │ │
│  │                                                             │ │
│  │ 4. MITIGATE                                                 │ │
│  │    • Apply immediate fix (rollback, failover)              │ │
│  │    • Reduce impact (rate limit, feature flag)              │ │
│  │                                                             │ │
│  │ 5. RESOLVE                                                  │ │
│  │    • Implement permanent fix                               │ │
│  │    • Verify resolution                                     │ │
│  │    • Close incident                                        │ │
│  │                                                             │ │
│  │ 6. REVIEW                                                   │ │
│  │    • Post-incident review                                  │ │
│  │    • Document learnings                                    │ │
│  │    • Create follow-up tasks                                │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  RUNBOOK EXAMPLE: High Replication Lag                         │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Symptoms:                                                   │ │
│  │ • Alert: pg_replication_lag_seconds > 30                   │ │
│  │ • Read replicas returning stale data                       │ │
│  │                                                             │ │
│  │ Diagnosis:                                                  │ │
│  │ 1. Check primary CPU/IO (overloaded?)                      │ │
│  │ 2. Check replica CPU/IO (can't keep up?)                   │ │
│  │ 3. Check network between primary/replica                   │ │
│  │ 4. Check for long-running transactions                     │ │
│  │ 5. Check for large batch operations                        │ │
│  │                                                             │ │
│  │ Mitigation:                                                 │ │
│  │ 1. Route traffic away from lagging replica                │ │
│  │ 2. Kill blocking transactions if safe                      │ │
│  │ 3. Scale up replica resources if needed                   │ │
│  │ 4. Pause batch jobs if causing lag                        │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```
