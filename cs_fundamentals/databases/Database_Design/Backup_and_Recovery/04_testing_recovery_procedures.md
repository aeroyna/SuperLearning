# Testing Recovery Procedures

## Why Test Recovery?

```
┌─────────────────────────────────────────────────────────────────┐
│              The Importance of Recovery Testing                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  "A backup that has never been tested is not a backup"          │
│                                                                  │
│  COMMON FAILURES DISCOVERED DURING TESTING                       │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ • Backup files corrupted or incomplete                    │ │
│  │ • Missing credentials for restore                         │ │
│  │ • Insufficient disk space at DR site                      │ │
│  │ • Network firewall blocking access                        │ │
│  │ • Wrong database version at DR site                       │ │
│  │ • Missing extensions or dependencies                      │ │
│  │ • Outdated runbooks and procedures                        │ │
│  │ • Staff unfamiliar with recovery process                  │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  TESTING BENEFITS                                                │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ ✓ Validates backup integrity                              │ │
│  │ ✓ Verifies RTO/RPO can be met                            │ │
│  │ ✓ Identifies gaps in procedures                          │ │
│  │ ✓ Trains team on recovery process                        │ │
│  │ ✓ Builds confidence for real incidents                   │ │
│  │ ✓ Satisfies compliance requirements                      │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Types of Recovery Tests

```
┌─────────────────────────────────────────────────────────────────┐
│              Recovery Test Categories                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  VERIFICATION TESTS (Automated, Frequent)                        │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Frequency: Daily or after each backup                     │ │
│  │ Scope: Backup file integrity                              │ │
│  │ Duration: Minutes                                          │ │
│  │                                                             │ │
│  │ Checks:                                                    │ │
│  │ • Backup file exists and is non-empty                     │ │
│  │ • Checksum verification                                   │ │
│  │ • Backup can be read/decompressed                         │ │
│  │ • WAL/binlog continuity                                   │ │
│  │                                                             │ │
│  │ Example Script:                                            │ │
│  │ ┌────────────────────────────────────────────────────────┐ │ │
│  │ │ #!/bin/bash                                            │ │ │
│  │ │ BACKUP="/backup/latest.dump"                           │ │ │
│  │ │                                                        │ │ │
│  │ │ # Check file exists and has size                      │ │ │
│  │ │ if [ ! -s "$BACKUP" ]; then                           │ │ │
│  │ │   echo "FAIL: Backup missing or empty"                │ │ │
│  │ │   exit 1                                               │ │ │
│  │ │ fi                                                     │ │ │
│  │ │                                                        │ │ │
│  │ │ # Verify checksum                                      │ │ │
│  │ │ sha256sum -c "$BACKUP.sha256" || exit 1               │ │ │
│  │ │                                                        │ │ │
│  │ │ # Test restore to null (PostgreSQL)                   │ │ │
│  │ │ pg_restore -l "$BACKUP" > /dev/null || exit 1         │ │ │
│  │ │                                                        │ │ │
│  │ │ echo "PASS: Backup verification successful"           │ │ │
│  │ └────────────────────────────────────────────────────────┘ │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  RESTORATION TESTS (Manual, Weekly/Monthly)                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Frequency: Weekly or monthly                              │ │
│  │ Scope: Full restore to test environment                   │ │
│  │ Duration: Hours                                            │ │
│  │                                                             │ │
│  │ Checks:                                                    │ │
│  │ • Complete restore succeeds                               │ │
│  │ • Data integrity verified                                 │ │
│  │ • Application can connect                                 │ │
│  │ • Performance acceptable                                  │ │
│  │                                                             │ │
│  │ Procedure:                                                 │ │
│  │ 1. Provision test database server                        │ │
│  │ 2. Restore from backup                                    │ │
│  │ 3. Run validation queries                                 │ │
│  │ 4. Test application connectivity                         │ │
│  │ 5. Document results                                       │ │
│  │ 6. Destroy test environment                               │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  FAILOVER TESTS (Planned, Quarterly)                             │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Frequency: Quarterly                                      │ │
│  │ Scope: Full failover to DR site                          │ │
│  │ Duration: Hours to days                                    │ │
│  │                                                             │ │
│  │ Types:                                                     │ │
│  │ • Tabletop: Walk through procedures without execution    │ │
│  │ • Parallel: Failover while primary remains available     │ │
│  │ • Full cutover: Production traffic to DR site            │ │
│  │                                                             │ │
│  │ Measures:                                                  │ │
│  │ • Time to detect failure                                  │ │
│  │ • Time to failover decision                               │ │
│  │ • Time to restore service                                 │ │
│  │ • Data loss (if any)                                      │ │
│  │ • Team coordination effectiveness                         │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  CHAOS ENGINEERING (Advanced)                                    │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ Intentionally inject failures in production               │ │
│  │                                                             │ │
│  │ Examples:                                                  │ │
│  │ • Kill primary database process                          │ │
│  │ • Disconnect network to primary                          │ │
│  │ • Fill disk on primary                                    │ │
│  │ • Corrupt data file (in test only!)                      │ │
│  │                                                             │ │
│  │ Prerequisites:                                             │ │
│  │ • Mature monitoring and alerting                          │ │
│  │ • Automatic failover capability                          │ │
│  │ • Strong rollback procedures                              │ │
│  │ • Executive approval                                      │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Automated Recovery Testing

```
┌─────────────────────────────────────────────────────────────────┐
│              Automated Test Implementation                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  FULL RESTORE TEST SCRIPT                                        │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ #!/bin/bash                                                │ │
│  │ set -e                                                     │ │
│  │                                                             │ │
│  │ # Configuration                                            │ │
│  │ BACKUP_FILE="/backup/latest.dump"                         │ │
│  │ TEST_DB="restore_test_$(date +%Y%m%d)"                    │ │
│  │ TEST_HOST="test-db-server.internal"                       │ │
│  │ EXPECTED_USERS=100000                                      │ │
│  │ EXPECTED_ORDERS=500000                                     │ │
│  │                                                             │ │
│  │ echo "Starting restore test at $(date)"                   │ │
│  │                                                             │ │
│  │ # Create test database                                     │ │
│  │ PGPASSWORD=$PGPASS psql -h $TEST_HOST -U postgres \       │ │
│  │   -c "DROP DATABASE IF EXISTS $TEST_DB;"                  │ │
│  │ PGPASSWORD=$PGPASS psql -h $TEST_HOST -U postgres \       │ │
│  │   -c "CREATE DATABASE $TEST_DB;"                          │ │
│  │                                                             │ │
│  │ # Time the restore                                         │ │
│  │ START_TIME=$(date +%s)                                     │ │
│  │ pg_restore -h $TEST_HOST -U postgres -d $TEST_DB \        │ │
│  │   --no-owner --no-acl $BACKUP_FILE                        │ │
│  │ END_TIME=$(date +%s)                                       │ │
│  │ RESTORE_DURATION=$((END_TIME - START_TIME))               │ │
│  │                                                             │ │
│  │ echo "Restore completed in $RESTORE_DURATION seconds"    │ │
│  │                                                             │ │
│  │ # Validate row counts                                      │ │
│  │ USERS=$(psql -h $TEST_HOST -U postgres -d $TEST_DB -t \   │ │
│  │   -c "SELECT COUNT(*) FROM users;")                       │ │
│  │ ORDERS=$(psql -h $TEST_HOST -U postgres -d $TEST_DB -t \  │ │
│  │   -c "SELECT COUNT(*) FROM orders;")                      │ │
│  │                                                             │ │
│  │ if [ $USERS -lt $EXPECTED_USERS ]; then                   │ │
│  │   echo "FAIL: Expected $EXPECTED_USERS users, got $USERS" │ │
│  │   exit 1                                                   │ │
│  │ fi                                                         │ │
│  │                                                             │ │
│  │ if [ $ORDERS -lt $EXPECTED_ORDERS ]; then                 │ │
│  │   echo "FAIL: Expected $EXPECTED_ORDERS orders"           │ │
│  │   exit 1                                                   │ │
│  │ fi                                                         │ │
│  │                                                             │ │
│  │ # Cleanup                                                  │ │
│  │ psql -h $TEST_HOST -U postgres \                          │ │
│  │   -c "DROP DATABASE $TEST_DB;"                            │ │
│  │                                                             │ │
│  │ # Report results                                           │ │
│  │ echo "SUCCESS: Restore test passed"                       │ │
│  │ echo "Users: $USERS, Orders: $ORDERS"                     │ │
│  │ echo "Duration: $RESTORE_DURATION seconds"                │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  CI/CD INTEGRATION (GitHub Actions)                              │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ name: Weekly Backup Test                                  │ │
│  │                                                             │ │
│  │ on:                                                        │ │
│  │   schedule:                                                │ │
│  │     - cron: '0 3 * * 0'  # Sunday 3 AM                   │ │
│  │                                                             │ │
│  │ jobs:                                                      │ │
│  │   test-restore:                                            │ │
│  │     runs-on: ubuntu-latest                                │ │
│  │     steps:                                                 │ │
│  │       - uses: actions/checkout@v3                         │ │
│  │                                                             │ │
│  │       - name: Download latest backup from S3              │ │
│  │         run: |                                             │ │
│  │           aws s3 cp s3://backups/latest.dump ./           │ │
│  │                                                             │ │
│  │       - name: Start PostgreSQL                            │ │
│  │         run: |                                             │ │
│  │           docker run -d --name testpg \                   │ │
│  │             -e POSTGRES_PASSWORD=test \                   │ │
│  │             -p 5432:5432 postgres:14                      │ │
│  │                                                             │ │
│  │       - name: Restore and validate                        │ │
│  │         run: ./scripts/restore-test.sh                    │ │
│  │                                                             │ │
│  │       - name: Notify on failure                           │ │
│  │         if: failure()                                      │ │
│  │         uses: slackapi/slack-github-action@v1             │ │
│  │         with:                                              │ │
│  │           channel: '#alerts'                              │ │
│  │           text: 'Backup restore test FAILED!'             │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Tabletop Exercises

```
┌─────────────────────────────────────────────────────────────────┐
│              Conducting Tabletop Exercises                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  WHAT IS A TABLETOP EXERCISE?                                    │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ A discussion-based session where team members walk        │ │
│  │ through a simulated disaster scenario step-by-step        │ │
│  │                                                             │ │
│  │ Benefits:                                                  │ │
│  │ • No production impact                                    │ │
│  │ • Tests procedures and decision-making                   │ │
│  │ • Identifies gaps in runbooks                             │ │
│  │ • Builds team familiarity                                 │ │
│  │ • Low cost to conduct                                     │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  SAMPLE SCENARIO                                                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ SCENARIO: Primary Database Corruption                     │ │
│  │                                                             │ │
│  │ Time 0: Monitoring detects high error rate               │ │
│  │ ─────────────────────────────────────────────────         │ │
│  │ Q: Who gets alerted first?                                │ │
│  │ Q: What's the first action?                               │ │
│  │ Q: How do we determine root cause?                        │ │
│  │                                                             │ │
│  │ Time 10min: Confirmed data corruption in orders table    │ │
│  │ ─────────────────────────────────────────────────         │ │
│  │ Q: Do we failover or attempt repair?                      │ │
│  │ Q: Who makes that decision?                               │ │
│  │ Q: How do we communicate to stakeholders?                 │ │
│  │                                                             │ │
│  │ Time 20min: Decision to restore from backup              │ │
│  │ ─────────────────────────────────────────────────         │ │
│  │ Q: Which backup do we use?                                │ │
│  │ Q: What's the expected data loss?                         │ │
│  │ Q: How long will restore take?                            │ │
│  │                                                             │ │
│  │ Time 45min: Restore completed                             │ │
│  │ ─────────────────────────────────────────────────         │ │
│  │ Q: How do we verify data integrity?                       │ │
│  │ Q: How do we handle missing transactions?                 │ │
│  │ Q: When do we declare recovery complete?                  │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  TABLETOP EXERCISE TEMPLATE                                      │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ 1. PREPARATION (Before meeting)                           │ │
│  │    □ Select scenario                                      │ │
│  │    □ Identify participants                                │ │
│  │    □ Prepare scenario details                             │ │
│  │    □ Gather current runbooks                              │ │
│  │    □ Schedule 2-hour meeting                              │ │
│  │                                                             │ │
│  │ 2. INTRODUCTION (15 min)                                  │ │
│  │    □ State objectives                                     │ │
│  │    □ Review ground rules                                  │ │
│  │    □ Introduce scenario                                   │ │
│  │                                                             │ │
│  │ 3. SCENARIO WALKTHROUGH (60 min)                          │ │
│  │    □ Present each phase                                   │ │
│  │    □ Discuss team actions                                 │ │
│  │    □ Note gaps and issues                                 │ │
│  │    □ Document decisions                                   │ │
│  │                                                             │ │
│  │ 4. DEBRIEF (30 min)                                       │ │
│  │    □ What went well?                                      │ │
│  │    □ What needs improvement?                              │ │
│  │    □ Action items                                         │ │
│  │    □ Timeline for fixes                                   │ │
│  │                                                             │ │
│  │ 5. FOLLOW-UP                                              │ │
│  │    □ Update runbooks                                      │ │
│  │    □ Address gaps                                         │ │
│  │    □ Schedule next exercise                               │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## DR Drill Execution

```
┌─────────────────────────────────────────────────────────────────┐
│              Full DR Drill Execution                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  PRE-DRILL CHECKLIST                                             │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ □ Schedule approved by management                         │ │
│  │ □ All stakeholders notified                               │ │
│  │ □ Rollback plan documented                                │ │
│  │ □ On-call team identified                                 │ │
│  │ □ Communication channels confirmed                        │ │
│  │ □ Success criteria defined                                │ │
│  │ □ Monitoring dashboards ready                             │ │
│  │ □ Backup of current state taken                           │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  DRILL PHASES                                                    │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │ PHASE 1: PREPARATION                                       │ │
│  │ ┌──────────────────────────────────────────────────────┐  │ │
│  │ │ T-1 day: Final checks                               │  │ │
│  │ │ • Verify DR site is ready                           │  │ │
│  │ │ • Confirm replication is current                    │  │ │
│  │ │ • Test monitoring alerts                            │  │ │
│  │ │ • Brief participants                                │  │ │
│  │ └──────────────────────────────────────────────────────┘  │ │
│  │                                                             │ │
│  │ PHASE 2: FAILOVER                                          │ │
│  │ ┌──────────────────────────────────────────────────────┐  │ │
│  │ │ T+0: Start drill                                    │  │ │
│  │ │ • Announce drill start                              │  │ │
│  │ │ • Start timer                                       │  │ │
│  │ │ • Execute failover procedures                       │  │ │
│  │ │                                                      │  │ │
│  │ │ # Record actual times for each step                 │  │ │
│  │ │ 10:00 - Drill announced                             │  │ │
│  │ │ 10:02 - Primary shutdown initiated                  │  │ │
│  │ │ 10:05 - DR promotion started                        │  │ │
│  │ │ 10:07 - DR promotion complete                       │  │ │
│  │ │ 10:10 - DNS updated                                 │  │ │
│  │ │ 10:12 - Application reconnected                     │  │ │
│  │ └──────────────────────────────────────────────────────┘  │ │
│  │                                                             │ │
│  │ PHASE 3: VALIDATION                                        │ │
│  │ ┌──────────────────────────────────────────────────────┐  │ │
│  │ │ • Run health checks                                 │  │ │
│  │ │ • Verify application functionality                  │  │ │
│  │ │ • Check data integrity                              │  │ │
│  │ │ • Monitor error rates                               │  │ │
│  │ │ • Confirm all services connected                    │  │ │
│  │ └──────────────────────────────────────────────────────┘  │ │
│  │                                                             │ │
│  │ PHASE 4: FAILBACK (if planned)                             │ │
│  │ ┌──────────────────────────────────────────────────────┐  │ │
│  │ │ • Resync primary from DR                            │  │ │
│  │ │ • Verify replication                                │  │ │
│  │ │ • Execute failback procedure                        │  │ │
│  │ │ • Validate original configuration                   │  │ │
│  │ └──────────────────────────────────────────────────────┘  │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  POST-DRILL REPORT                                               │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │ DR DRILL REPORT                                            │ │
│  │ Date: 2024-01-15                                           │ │
│  │                                                             │ │
│  │ METRICS                                                    │ │
│  │ ────────────────────────────────────────────               │ │
│  │ Target RTO: 30 minutes                                    │ │
│  │ Actual RTO: 22 minutes ✓                                  │ │
│  │                                                             │ │
│  │ Target RPO: 5 minutes                                     │ │
│  │ Actual RPO: 2 minutes ✓                                   │ │
│  │                                                             │ │
│  │ TIMELINE                                                   │ │
│  │ ────────────────────────────────────────────               │ │
│  │ 10:00 - Drill start                                       │ │
│  │ 10:05 - Failover initiated                                │ │
│  │ 10:12 - DR accepting traffic                              │ │
│  │ 10:22 - Full recovery confirmed                           │ │
│  │                                                             │ │
│  │ ISSUES FOUND                                               │ │
│  │ ────────────────────────────────────────────               │ │
│  │ 1. DNS TTL too long (5 min vs 1 min expected)            │ │
│  │ 2. One service had hardcoded IP                          │ │
│  │ 3. Runbook step 7 was out of date                        │ │
│  │                                                             │ │
│  │ ACTION ITEMS                                               │ │
│  │ ────────────────────────────────────────────               │ │
│  │ □ Reduce DNS TTL to 60 seconds                           │ │
│  │ □ Update service X to use DNS name                       │ │
│  │ □ Update runbook with current procedure                  │ │
│  │                                                             │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

## Testing Schedule

```
┌─────────────────────────────────────────────────────────────────┐
│              Recommended Testing Schedule                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  TESTING FREQUENCY                                               │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │ Test Type          │ Frequency    │ Owner    │ Duration   │ │
│  │ ───────────────────┼──────────────┼──────────┼─────────── │ │
│  │ Backup verification│ Daily        │ Automated│ 5 min      │ │
│  │ Restore test       │ Weekly       │ DBA      │ 2 hours    │ │
│  │ PITR test          │ Monthly      │ DBA      │ 4 hours    │ │
│  │ Tabletop exercise  │ Quarterly    │ DR Lead  │ 2 hours    │ │
│  │ Failover drill     │ Semi-annual  │ DR Lead  │ 4-8 hours  │ │
│  │ Full DR test       │ Annual       │ Director │ 1-2 days   │ │
│  │                                                             │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ANNUAL CALENDAR                                                 │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │ Month │ Activity                                           │ │
│  │ ──────┼──────────────────────────────────────────────────  │ │
│  │ Jan   │ Full DR test                                       │ │
│  │ Feb   │ Review and update procedures                      │ │
│  │ Mar   │ Tabletop exercise                                 │ │
│  │ Apr   │ Failover drill (planned)                          │ │
│  │ May   │ Review DR costs and architecture                  │ │
│  │ Jun   │ Tabletop exercise                                 │ │
│  │ Jul   │ Training for new team members                     │ │
│  │ Aug   │ Failover drill (unannounced)                      │ │
│  │ Sep   │ Tabletop exercise                                 │ │
│  │ Oct   │ Update RTO/RPO requirements                       │ │
│  │ Nov   │ Tabletop exercise                                 │ │
│  │ Dec   │ Annual DR plan review                             │ │
│  │                                                             │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  COMPLIANCE REQUIREMENTS                                         │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │                                                             │ │
│  │ Standard   │ Testing Requirement                           │ │
│  │ ───────────┼────────────────────────────────────────────── │ │
│  │ SOC 2      │ Annual DR testing with documented results    │ │
│  │ PCI-DSS    │ Annual testing, quarterly reviews            │ │
│  │ HIPAA      │ Regular testing, documented procedures       │ │
│  │ ISO 27001  │ Regular testing per risk assessment          │ │
│  │ GDPR       │ Ability to restore data upon request         │ │
│  │                                                             │ │
│  │ Always document:                                           │ │
│  │ • Test date and participants                              │ │
│  │ • Scenarios tested                                        │ │
│  │ • Results and metrics                                     │ │
│  │ • Issues found and remediation                            │ │
│  │                                                             │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```
