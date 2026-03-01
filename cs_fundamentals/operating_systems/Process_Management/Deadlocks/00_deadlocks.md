# Deadlocks

## Overview

Deadlock is a situation where a set of processes are blocked because each process is holding a resource and waiting for another resource acquired by some other process. Understanding deadlock is critical for interview preparation.

## Topics Covered

1. **[Deadlock Characterization](01_deadlock_characterization.md)**
   - What is deadlock?
   - Four necessary conditions (Mutual Exclusion, Hold and Wait, No Preemption, Circular Wait)
   - System model
   - Resource allocation scenarios

2. **[Resource Allocation Graph](02_resource_allocation_graph.md)**
   - Graph representation
   - Request edge and assignment edge
   - Detecting cycles
   - Graph with and without deadlock
   - Claim edges

3. **[Deadlock Prevention](03_deadlock_prevention.md)**
   - Eliminate mutual exclusion
   - Eliminate hold and wait
   - Allow preemption
   - Eliminate circular wait
   - Practical approaches

4. **[Deadlock Avoidance (Banker's Algorithm)](04_deadlock_avoidance.md)**
   - Safe state vs unsafe state
   - Resource allocation algorithm
   - Safety algorithm
   - Banker's algorithm for single and multiple resource instances
   - Limitations

5. **[Deadlock Detection and Recovery](05_deadlock_detection_recovery.md)**
   - When to check for deadlock
   - Detection algorithm
   - Recovery methods (process termination, resource preemption)
   - Selecting a victim
   - Starvation

## Key Takeaways

- Deadlock requires all four conditions simultaneously
- Prevention: Break one of the four conditions
- Avoidance: Don't enter unsafe state (Banker's algorithm)
- Detection: Allow deadlock, then detect and recover
- Real systems often use ostrich algorithm (ignore deadlock)

## Interview Focus

- State and explain four necessary conditions
- Draw and interpret resource allocation graphs
- Solve Banker's algorithm problems
- Compare prevention, avoidance, and detection
- Understand why some systems ignore deadlock
