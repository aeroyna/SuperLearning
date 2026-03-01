# Disaster Recovery Planning

## Disaster Recovery Overview

```
┌─────────────────────────────────────────────────────────────────┐
│              Disaster Recovery Fundamentals                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  WHAT IS DISASTER RECOVERY?                                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ A structured approach to restoring critical IT systems    │ │
│  │ and data after a catastrophic event                       │ │
│  │                                                             │ │
│  │ Goals:                                                     │ │
│  │ • Minimize downtime                                        │ │
│  │ • Minimize data loss                                       │ │
│  │ • Ensure business continuity                              │ │
│  │ • Protect against various failure scenarios               │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  TYPES OF DISASTERS                                              │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  LOCAL FAILURES           │  REGIONAL DISASTERS            │ │
│  │  ─────────────────────────┼──────────────────────────────  │ │
│  │  • Hardware failure       │  • Natural disasters          │ │
│  │  • Software bugs          │  • Power grid failure         │ │
│  │  • Human error            │  • Network outage             │ │
│  │  • Ransomware             │  • Data center destruction    │ │
│  │                           │                                │ │
│  │  Recovery: Local backup   │  Recovery: Offsite/DR site   │ │
│  │                                                             │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  DR TIERS                                                        │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │ Tier 0: No offsite data                                   │ │
│  │ ───────────────────────                                    │ │
│  │ • Just local backups                                       │ │
│  │ • RTO: Days to weeks                                       │ │
│  │ • RPO: Last backup (hours to days)                        │ │
│  │                                                             │ │
│  │ Tier 1: Offsite backup                                    │ │
│  │ ───────────────────────                                    │ │
│  │ • PITR backups stored offsite                             │ │
│  │ • RTO: Hours                                               │ │
│  │ • RPO: Minutes to hours                                    │ │
│  │                                                             │ │
│  │ Tier 2: Warm standby                                      │ │
│  │ ───────────────────────                                    │ │
│  │ • Async replica at DR site                                │ │
│  │ • RTO: Minutes to hours                                    │ │
│  │ • RPO: Seconds to minutes                                  │ │
│  │                                                             │ │
│  │ Tier 3: Hot standby                                       │ │
│  │ ───────────────────────                                    │ │
│  │ • Sync replica with auto-failover                         │ │
│  │ • RTO: Seconds                                             │ │
│  │ • RPO: Zero (no data loss)                                │ │
│  │                                                             │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## DR Architecture Patterns

```
┌─────────────────────────────────────────────────────────────────┐
│              Disaster Recovery Architectures                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  BACKUP AND RESTORE                                              │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  Region A (Primary)           Region B (DR)               │ │
│  │  ┌─────────────────┐                                       │ │
│  │  │    Database     │                                       │ │
│  │  └────────┬────────┘                                       │ │
│  │           │ backup                                         │ │
│  │           ▼                                                │ │
│  │  ┌─────────────────┐    replicate    ┌─────────────────┐  │ │
│  │  │  Object Storage │ ──────────────► │  Object Storage │  │ │
│  │  └─────────────────┘                 └─────────────────┘  │ │
│  │                                                             │ │
│  │  RTO: Hours │ RPO: Last backup │ Cost: $                  │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  PILOT LIGHT                                                     │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  Region A (Primary)           Region B (DR)               │ │
│  │  ┌─────────────────┐          ┌─────────────────┐         │ │
│  │  │    Database     │ ───────► │ Minimal Replica │         │ │
│  │  │   (Running)     │  async   │   (Running)     │         │ │
│  │  └─────────────────┘          └─────────────────┘         │ │
│  │  ┌─────────────────┐          ┌─────────────────┐         │ │
│  │  │   App Servers   │          │ AMIs/Images     │         │ │
│  │  │   (Running)     │          │ (Not running)   │         │ │
│  │  └─────────────────┘          └─────────────────┘         │ │
│  │                                                             │ │
│  │  RTO: 10-30 min │ RPO: Seconds │ Cost: $$                 │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  WARM STANDBY                                                    │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  Region A (Primary)           Region B (DR)               │ │
│  │  ┌─────────────────┐          ┌─────────────────┐         │ │
│  │  │    Database     │ ───────► │   Hot Replica   │         │ │
│  │  │   (Running)     │  async   │   (Running)     │         │ │
│  │  └─────────────────┘          └─────────────────┘         │ │
│  │  ┌─────────────────┐          ┌─────────────────┐         │ │
│  │  │   App Servers   │          │ App Servers     │         │ │
│  │  │   (Full cap)    │          │ (Min capacity)  │         │ │
│  │  └─────────────────┘          └─────────────────┘         │ │
│  │                                                             │ │
│  │  RTO: Minutes │ RPO: Seconds │ Cost: $$$                  │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ACTIVE-ACTIVE (MULTI-REGION)                                    │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  Region A                     Region B                     │ │
│  │  ┌─────────────────┐          ┌─────────────────┐         │ │
│  │  │    Database     │ ◄──────► │    Database     │         │ │
│  │  │   (Primary)     │   sync   │   (Primary)     │         │ │
│  │  └─────────────────┘          └─────────────────┘         │ │
│  │  ┌─────────────────┐          ┌─────────────────┐         │ │
│  │  │   App Servers   │          │   App Servers   │         │ │
│  │  │   (Full cap)    │          │   (Full cap)    │         │ │
│  │  └─────────────────┘          └─────────────────┘         │ │
│  │          │                            │                    │ │
│  │          └──────────┬─────────────────┘                    │ │
│  │                     ▼                                      │ │
│  │            Global Load Balancer                            │ │
│  │                                                             │ │
│  │  RTO: 0 │ RPO: 0 │ Cost: $$$$                             │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Database Replication for DR

```
┌─────────────────────────────────────────────────────────────────┐
│              Replication Strategies for DR                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ASYNCHRONOUS REPLICATION                                        │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  Primary ────────────────────► Standby                     │ │
│  │     │                              │                        │ │
│  │     │ Commit returns               │ Apply changes         │ │
│  │     │ immediately                  │ later                  │ │
│  │     ▼                              ▼                        │ │
│  │  Write 1 ───────► Write 2     Write 1 (delayed)           │ │
│  │                                                             │ │
│  │  + No latency impact on primary                            │ │
│  │  + Works over high-latency links                          │ │
│  │  - Potential data loss on failover                        │ │
│  │  - Lag can vary                                            │ │
│  │                                                             │ │
│  │  PostgreSQL:                                               │ │
│  │  synchronous_commit = off                                  │ │
│  │                                                             │ │
│  │  MySQL:                                                    │ │
│  │  CHANGE MASTER TO ... (default is async)                  │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  SYNCHRONOUS REPLICATION                                         │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  Primary ────────────────────► Standby                     │ │
│  │     │                              │                        │ │
│  │     │ Commit waits                 │ Confirm               │ │
│  │     │ for confirmation             │ received              │ │
│  │     │◄─────────────────────────────│                        │ │
│  │     ▼                              ▼                        │ │
│  │  Write 1 committed            Write 1 durable              │ │
│  │                                                             │ │
│  │  + Zero data loss                                          │ │
│  │  + Guaranteed consistency                                  │ │
│  │  - Higher commit latency                                   │ │
│  │  - Requires low-latency link                              │ │
│  │                                                             │ │
│  │  PostgreSQL:                                               │ │
│  │  synchronous_commit = on                                   │ │
│  │  synchronous_standby_names = 'dr_standby'                 │ │
│  │                                                             │ │
│  │  MySQL (Semi-sync):                                        │ │
│  │  rpl_semi_sync_master_enabled = 1                         │ │
│  │  rpl_semi_sync_slave_enabled = 1                          │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  CROSS-REGION REPLICATION                                        │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  AWS RDS Cross-Region Read Replica:                       │ │
│  │  ┌─────────────────────────────────────────────────────┐  │ │
│  │  │ aws rds create-db-instance-read-replica \           │  │ │
│  │  │   --db-instance-identifier dr-replica \             │  │ │
│  │  │   --source-db-instance-identifier prod-db \         │  │ │
│  │  │   --source-region us-east-1 \                       │  │ │
│  │  │   --region us-west-2                                │  │ │
│  │  └─────────────────────────────────────────────────────┘  │ │
│  │                                                             │ │
│  │  PostgreSQL Streaming Replication:                        │ │
│  │  ┌─────────────────────────────────────────────────────┐  │ │
│  │  │ # primary postgresql.conf                           │  │ │
│  │  │ listen_addresses = '*'                              │  │ │
│  │  │ wal_level = replica                                 │  │ │
│  │  │ max_wal_senders = 5                                 │  │ │
│  │  │ wal_keep_size = 1GB                                 │  │ │
│  │  │                                                      │  │ │
│  │  │ # standby (recovery.conf or postgresql.auto.conf)  │  │ │
│  │  │ primary_conninfo = 'host=primary.example.com ...'  │  │ │
│  │  └─────────────────────────────────────────────────────┘  │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Failover Procedures

```
┌─────────────────────────────────────────────────────────────────┐
│              Failover Procedures                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  MANUAL FAILOVER                                                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  Step 1: Confirm primary is down                          │ │
│  │  ┌──────────────────────────────────────────────────────┐ │ │
│  │  │ • Verify from multiple sources                       │ │ │
│  │  │ • Check monitoring alerts                            │ │ │
│  │  │ • Attempt connection to primary                      │ │ │
│  │  │ • Confirm not a network partition                    │ │ │
│  │  └──────────────────────────────────────────────────────┘ │ │
│  │                                                             │ │
│  │  Step 2: Promote standby                                   │ │
│  │  ┌──────────────────────────────────────────────────────┐ │ │
│  │  │ # PostgreSQL                                         │ │ │
│  │  │ pg_ctl promote -D /var/lib/postgresql/14/main       │ │ │
│  │  │ # Or                                                 │ │ │
│  │  │ SELECT pg_promote();                                 │ │ │
│  │  │                                                      │ │ │
│  │  │ # MySQL                                              │ │ │
│  │  │ STOP SLAVE;                                          │ │ │
│  │  │ RESET SLAVE ALL;                                     │ │ │
│  │  └──────────────────────────────────────────────────────┘ │ │
│  │                                                             │ │
│  │  Step 3: Update DNS/connection strings                    │ │
│  │  ┌──────────────────────────────────────────────────────┐ │ │
│  │  │ • Update DNS records (if using DNS failover)        │ │ │
│  │  │ • Update application connection strings             │ │ │
│  │  │ • Update load balancer targets                      │ │ │
│  │  │ • Invalidate connection pools                       │ │ │
│  │  └──────────────────────────────────────────────────────┘ │ │
│  │                                                             │ │
│  │  Step 4: Verify and monitor                               │ │
│  │  ┌──────────────────────────────────────────────────────┐ │ │
│  │  │ • Test application connectivity                     │ │ │
│  │  │ • Verify data integrity                             │ │ │
│  │  │ • Monitor performance                               │ │ │
│  │  │ • Alert team of failover completion                 │ │ │
│  │  └──────────────────────────────────────────────────────┘ │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  AUTOMATIC FAILOVER                                              │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  Patroni (PostgreSQL):                                    │ │
│  │  ┌──────────────────────────────────────────────────────┐ │ │
│  │  │ # patroni.yml                                        │ │ │
│  │  │ scope: postgres-cluster                              │ │ │
│  │  │ name: node1                                          │ │ │
│  │  │                                                      │ │ │
│  │  │ restapi:                                             │ │ │
│  │  │   listen: 0.0.0.0:8008                              │ │ │
│  │  │                                                      │ │ │
│  │  │ etcd:                                                │ │ │
│  │  │   hosts: etcd1:2379,etcd2:2379,etcd3:2379           │ │ │
│  │  │                                                      │ │ │
│  │  │ bootstrap:                                           │ │ │
│  │  │   dcs:                                               │ │ │
│  │  │     ttl: 30                                         │ │ │
│  │  │     loop_wait: 10                                   │ │ │
│  │  │     retry_timeout: 10                               │ │ │
│  │  │     maximum_lag_on_failover: 1048576                │ │ │
│  │  │                                                      │ │ │
│  │  │ postgresql:                                          │ │ │
│  │  │   listen: 0.0.0.0:5432                              │ │ │
│  │  │   data_dir: /var/lib/postgresql/14/main             │ │ │
│  │  └──────────────────────────────────────────────────────┘ │ │
│  │                                                             │ │
│  │  MySQL Group Replication:                                  │ │
│  │  ┌──────────────────────────────────────────────────────┐ │ │
│  │  │ # my.cnf                                             │ │ │
│  │  │ [mysqld]                                             │ │ │
│  │  │ plugin_load_add='group_replication.so'              │ │ │
│  │  │ group_replication_group_name="uuid-here"            │ │ │
│  │  │ group_replication_start_on_boot=off                 │ │ │
│  │  │ group_replication_local_address="node1:33061"       │ │ │
│  │  │ group_replication_group_seeds="node1:33061,..."     │ │ │
│  │  │ group_replication_single_primary_mode=ON            │ │ │
│  │  └──────────────────────────────────────────────────────┘ │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## DR Plan Documentation

```
┌─────────────────────────────────────────────────────────────────┐
│              DR Plan Template                                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. EXECUTIVE SUMMARY                                            │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Purpose: Restore database operations after disaster     │ │
│  │ • Scope: Production database systems                      │ │
│  │ • RTO Target: 1 hour                                      │ │
│  │ • RPO Target: 15 minutes                                  │ │
│  │ • Last Updated: [Date]                                    │ │
│  │ • Plan Owner: [Name]                                      │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  2. CONTACT LIST                                                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Role              │ Name       │ Phone        │ Email     │ │
│  │ ──────────────────┼────────────┼──────────────┼────────── │ │
│  │ DR Lead           │ [Name]     │ [Phone]      │ [Email]   │ │
│  │ DBA Primary       │ [Name]     │ [Phone]      │ [Email]   │ │
│  │ DBA Backup        │ [Name]     │ [Phone]      │ [Email]   │ │
│  │ Infrastructure    │ [Name]     │ [Phone]      │ [Email]   │ │
│  │ Application Lead  │ [Name]     │ [Phone]      │ [Email]   │ │
│  │ Management        │ [Name]     │ [Phone]      │ [Email]   │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  3. SYSTEM INVENTORY                                             │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ System         │ Type    │ Priority │ DR Location         │ │
│  │ ───────────────┼─────────┼──────────┼──────────────────── │ │
│  │ prod-db-1      │ Primary │ Critical │ us-west-2           │ │
│  │ prod-db-2      │ Replica │ High     │ us-east-1           │ │
│  │ analytics-db   │ Primary │ Medium   │ Object storage      │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  4. RECOVERY PROCEDURES                                          │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Scenario: Complete Primary Failure                        │ │
│  │                                                             │ │
│  │ □ Step 1: Confirm failure (5 min)                         │ │
│  │   - Check monitoring dashboards                           │ │
│  │   - Verify from multiple sources                          │ │
│  │   - Contact cloud provider if needed                      │ │
│  │                                                             │ │
│  │ □ Step 2: Activate DR site (10 min)                       │ │
│  │   - Promote DR replica                                    │ │
│  │   - Verify replica is healthy                             │ │
│  │   - Run: pg_ctl promote -D /data                         │ │
│  │                                                             │ │
│  │ □ Step 3: Update routing (5 min)                          │ │
│  │   - Update DNS: db.example.com → DR IP                   │ │
│  │   - Update application configs                            │ │
│  │   - Restart connection pools                              │ │
│  │                                                             │ │
│  │ □ Step 4: Verify recovery (10 min)                        │ │
│  │   - Test application connectivity                         │ │
│  │   - Run data integrity checks                             │ │
│  │   - Monitor error rates                                   │ │
│  │                                                             │ │
│  │ □ Step 5: Notify stakeholders                             │ │
│  │   - Send recovery notification                            │ │
│  │   - Update status page                                    │ │
│  │   - Schedule post-mortem                                  │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  5. RECOVERY VERIFICATION CHECKLIST                              │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ □ Database accepting connections                          │ │
│  │ □ All critical tables accessible                          │ │
│  │ □ Row counts match expected values                        │ │
│  │ □ Application can read and write                          │ │
│  │ □ Replication lag is acceptable                           │ │
│  │ □ No errors in database logs                              │ │
│  │ □ Performance metrics normal                              │ │
│  │ □ Backups configured for new primary                      │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## DR Cost Considerations

```
┌─────────────────────────────────────────────────────────────────┐
│              DR Cost vs Recovery Time                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  COST-RECOVERY TRADEOFF                                          │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │  Cost ▲                                                    │ │
│  │       │                        ┌────── Active-Active      │ │
│  │       │                    ────┘                           │ │
│  │       │               ────┘          Hot Standby           │ │
│  │       │          ────┘                                     │ │
│  │       │     ────┘                    Warm Standby          │ │
│  │       │────┘                                               │ │
│  │       │                              Pilot Light           │ │
│  │       │    Backup Only                                     │ │
│  │       └────────────────────────────────────────────► RTO   │ │
│  │         0        Minutes      Hours        Days            │ │
│  │                                                             │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  COST OPTIMIZATION STRATEGIES                                    │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │ Reserved Instances for DR:                                 │ │
│  │ • Use reserved/committed capacity for always-on DR        │ │
│  │ • Significant savings vs on-demand                        │ │
│  │                                                             │ │
│  │ Smaller DR Environment:                                    │ │
│  │ • Scale up DR during failover                             │ │
│  │ • Run DR at 50% capacity normally                         │ │
│  │                                                             │ │
│  │ Multi-Use DR:                                              │ │
│  │ • Use DR for read replicas during normal ops             │ │
│  │ • Use for analytics/reporting                             │ │
│  │ • Use for dev/test (with caution)                         │ │
│  │                                                             │ │
│  │ Tiered Storage:                                            │ │
│  │ • Glacier for long-term backup archives                   │ │
│  │ • Standard storage for recent backups                     │ │
│  │                                                             │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  MONTHLY COST EXAMPLES (Approximate)                             │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │ Strategy          │ Monthly Cost │ RTO      │ RPO          │ │
│  │ ──────────────────┼──────────────┼──────────┼───────────── │ │
│  │ S3 Backups Only   │ $100-500     │ 4-8 hrs  │ Last backup │ │
│  │ Pilot Light       │ $200-1000    │ 30 min   │ Minutes     │ │
│  │ Warm Standby      │ $1000-5000   │ 10 min   │ Seconds     │ │
│  │ Hot Standby       │ $3000-10000  │ <1 min   │ Zero        │ │
│  │ Active-Active     │ $10000+      │ 0        │ Zero        │ │
│  │                                                             │ │
│  │ (Costs vary by database size and cloud provider)          │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```
