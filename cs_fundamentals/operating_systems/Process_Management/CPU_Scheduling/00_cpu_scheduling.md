# CPU Scheduling

## Overview

CPU scheduling determines which process runs when there are multiple runnable processes. This is core to multiprogramming and time-sharing systems.

## Topics Covered

1. **[Scheduling Criteria](01_scheduling_criteria.md)**
   - CPU utilization
   - Throughput
   - Turnaround time
   - Waiting time
   - Response time
   - Fairness

2. **[FCFS Scheduling](02_fcfs_scheduling.md)**
   - First-Come, First-Served
   - Non-preemptive
   - Convoy effect
   - Average waiting time calculation
   - Advantages and disadvantages

3. **[SJF Scheduling](03_sjf_scheduling.md)**
   - Shortest Job First
   - Preemptive (SRTF) and non-preemptive versions
   - Optimal average waiting time
   - Starvation problem
   - Predicting next CPU burst

4. **[Priority Scheduling](04_priority_scheduling.md)**
   - Priority assignment
   - Preemptive vs non-preemptive
   - Starvation and aging solution
   - Priority inversion

5. **[Round Robin Scheduling](05_round_robin_scheduling.md)**
   - Time quantum/time slice
   - Preemptive scheduling
   - Context switching overhead
   - Effect of time quantum size
   - Fair CPU allocation

6. **[Multilevel Queue Scheduling](06_multilevel_queue_scheduling.md)**
   - Multiple ready queues
   - Queue priorities
   - Multilevel feedback queue
   - Adaptive scheduling
   - Example: Unix/Linux scheduler

7. **[Real-Time Scheduling](07_real_time_scheduling.md)**
   - Hard vs soft real-time
   - Rate Monotonic Scheduling (RMS)
   - Earliest Deadline First (EDF)
   - Schedulability analysis

## Key Takeaways

- Different algorithms optimize different criteria
- No single best algorithm for all scenarios
- Trade-off between fairness, turnaround time, and overhead
- Real-time systems have strict timing constraints

## Interview Focus

- Calculate average waiting time for different algorithms
- Compare FCFS, SJF, Priority, and Round Robin
- Understand convoy effect and starvation
- Explain how time quantum affects Round Robin
- Multilevel feedback queue as used in modern OS
