# Backup and Disaster Recovery

## Introduction

Database backup and disaster recovery are critical components of any production database system. A well-designed backup strategy protects against data loss from hardware failures, human errors, security breaches, and natural disasters.

## Topics in This Section

1. **[Backup Strategies](01_backup_strategies.md)**
2. **[Point-in-Time Recovery](02_point_in_time_recovery.md)**
3. **[Disaster Recovery Planning](03_disaster_recovery_planning.md)**
4. **[Testing Recovery Procedures](04_testing_recovery_procedures.md)**

## Why Backups Matter

```
┌─────────────────────────────────────────────────────────────────┐
│              Common Causes of Data Loss                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  HARDWARE FAILURES                                               │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Disk failures                                            │ │
│  │ • RAID controller failures                                 │ │
│  │ • Power supply failures                                    │ │
│  │ • Memory corruption                                        │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  HUMAN ERRORS                                                    │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Accidental DELETE/DROP                                   │ │
│  │ • Wrong UPDATE without WHERE                               │ │
│  │ • Misconfigured migrations                                 │ │
│  │ • Incorrect data imports                                   │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  SECURITY INCIDENTS                                              │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Ransomware attacks                                       │ │
│  │ • SQL injection leading to data destruction               │ │
│  │ • Malicious insiders                                       │ │
│  │ • Compromised credentials                                  │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  DISASTERS                                                       │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Natural disasters (flood, fire, earthquake)             │ │
│  │ • Data center outages                                      │ │
│  │ • Regional power failures                                  │ │
│  │ • Network failures                                         │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Key Concepts

```
┌─────────────────────────────────────────────────────────────────┐
│              Backup and Recovery Terms                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  RPO (Recovery Point Objective)                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ "How much data can we afford to lose?"                     │ │
│  │                                                             │ │
│  │ Last Backup         Failure                                │ │
│  │     │                  │                                   │ │
│  │     ▼                  ▼                                   │ │
│  │ ────●──────────────────●────────►                         │ │
│  │     │←───── RPO ──────→│                                   │ │
│  │                                                             │ │
│  │ • RPO = 1 hour → max 1 hour of data loss                  │ │
│  │ • RPO = 0 → zero data loss (synchronous replication)      │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  RTO (Recovery Time Objective)                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ "How long can we be down?"                                 │ │
│  │                                                             │ │
│  │ Failure          Recovery Complete                         │ │
│  │     │                  │                                   │ │
│  │     ▼                  ▼                                   │ │
│  │ ────●──────────────────●────────►                         │ │
│  │     │←───── RTO ──────→│                                   │ │
│  │                                                             │ │
│  │ • RTO = 4 hours → must be back online in 4 hours          │ │
│  │ • RTO = 0 → instant failover (hot standby)                │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  BACKUP TYPES                                                    │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Full Backup:                                               │ │
│  │   Complete copy of all data                                │ │
│  │   Large, slow, but simplest to restore                    │ │
│  │                                                             │ │
│  │ Incremental Backup:                                        │ │
│  │   Only changes since last backup                          │ │
│  │   Small, fast, requires chain for restore                 │ │
│  │                                                             │ │
│  │ Differential Backup:                                       │ │
│  │   Changes since last full backup                          │ │
│  │   Medium size, simpler restore than incremental           │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Backup Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│              Backup Architecture                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────┐                                                │
│  │  Production │                                                │
│  │   Database  │                                                │
│  └──────┬──────┘                                                │
│         │                                                        │
│         ├────────────────────┬─────────────────────┐            │
│         │                    │                     │            │
│         ▼                    ▼                     ▼            │
│  ┌─────────────┐     ┌─────────────┐      ┌─────────────┐      │
│  │  Streaming  │     │   Logical   │      │  Snapshot   │      │
│  │ Replication │     │   Backups   │      │   Backups   │      │
│  └──────┬──────┘     └──────┬──────┘      └──────┬──────┘      │
│         │                    │                     │            │
│         ▼                    ▼                     ▼            │
│  ┌─────────────┐     ┌─────────────┐      ┌─────────────┐      │
│  │   Standby   │     │  Backup     │      │   Object    │      │
│  │   Server    │     │  Server     │      │   Storage   │      │
│  └─────────────┘     └──────┬──────┘      └──────┬──────┘      │
│                             │                     │             │
│                             └──────────┬──────────┘             │
│                                        │                        │
│                                        ▼                        │
│                             ┌─────────────────────┐             │
│                             │   Offsite Storage   │             │
│                             │   (Different Region)│             │
│                             └─────────────────────┘             │
└─────────────────────────────────────────────────────────────────┘
```

## The 3-2-1 Backup Rule

```
┌─────────────────────────────────────────────────────────────────┐
│              3-2-1 Backup Strategy                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  3 - THREE COPIES OF DATA                                        │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐        │ │
│  │ │  Production  │ │   Backup 1   │ │   Backup 2   │        │ │
│  │ │    (Live)    │ │   (Recent)   │ │  (Archive)   │        │ │
│  │ └──────────────┘ └──────────────┘ └──────────────┘        │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  2 - TWO DIFFERENT STORAGE TYPES                                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ ┌──────────────┐              ┌──────────────┐             │ │
│  │ │  Local Disk  │              │ Object Store │             │ │
│  │ │   (NVMe/SSD) │              │   (S3/GCS)   │             │ │
│  │ └──────────────┘              └──────────────┘             │ │
│  │                                                             │ │
│  │ Protects against media-specific failures                   │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  1 - ONE OFFSITE COPY                                            │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ ┌──────────────┐              ┌──────────────┐             │ │
│  │ │  Region A    │ ──────────── │  Region B    │             │ │
│  │ │  (Primary)   │   Replicate  │  (DR Site)   │             │ │
│  │ └──────────────┘              └──────────────┘             │ │
│  │                                                             │ │
│  │ Protects against regional disasters                        │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Backup Checklist

```
┌─────────────────────────────────────────────────────────────────┐
│              Backup Implementation Checklist                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  PLANNING                                                        │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ □ Define RPO and RTO requirements                          │ │
│  │ □ Document backup strategy                                 │ │
│  │ □ Choose backup tools                                      │ │
│  │ □ Plan storage locations                                   │ │
│  │ □ Calculate storage requirements                           │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  IMPLEMENTATION                                                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ □ Configure automated backups                              │ │
│  │ □ Set up monitoring and alerts                             │ │
│  │ □ Implement backup encryption                              │ │
│  │ □ Configure retention policies                             │ │
│  │ □ Set up offsite replication                               │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  VALIDATION                                                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ □ Test restore procedures                                  │ │
│  │ □ Verify backup integrity                                  │ │
│  │ □ Document recovery procedures                             │ │
│  │ □ Train operations team                                    │ │
│  │ □ Schedule regular DR drills                               │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  MAINTENANCE                                                     │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ □ Monitor backup success/failure                           │ │
│  │ □ Review storage usage                                     │ │
│  │ □ Update procedures as needed                              │ │
│  │ □ Quarterly recovery tests                                 │ │
│  │ □ Annual DR plan review                                    │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```
