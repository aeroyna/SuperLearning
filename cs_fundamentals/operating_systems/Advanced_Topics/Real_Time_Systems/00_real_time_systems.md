# Real-Time Systems

## Overview

Real-time systems have strict timing constraints. Understanding real-time OS concepts is important for embedded systems and critical applications.

## Topics Covered

1. **[Real-Time System Characteristics](01_rt_characteristics.md)**
   - What is a real-time system?
   - Hard real-time vs soft real-time
   - Determinism and responsiveness
   - Real-time constraints
   - Application examples (automotive, medical, aerospace)

2. **[Real-Time Scheduling Algorithms](02_rt_scheduling.md)**
   - Rate Monotonic Scheduling (RMS)
   - Earliest Deadline First (EDF)
   - Deadline Monotonic Scheduling
   - Schedulability analysis
   - Liu and Layland bound

3. **[Priority Inversion](03_priority_inversion.md)**
   - What is priority inversion?
   - Unbounded priority inversion
   - Priority inheritance protocol
   - Priority ceiling protocol
   - Mars Pathfinder incident

## Key Takeaways

- Real-time systems must meet timing deadlines
- Hard real-time: missing deadline is catastrophic
- Soft real-time: occasional deadline misses tolerable
- Priority inversion can cause high-priority tasks to miss deadlines

## Interview Focus

- Distinguish hard vs soft real-time
- Understand RMS and EDF scheduling
- Explain priority inversion and solutions
- Know real-world examples
