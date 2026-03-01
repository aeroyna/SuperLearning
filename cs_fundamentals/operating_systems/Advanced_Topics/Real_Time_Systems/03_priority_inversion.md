# Priority Inversion\n\n## Priority Inversion

### Overview

Priority inheritance, priority ceiling protocols

### Characteristics

Real-time systems must meet strict timing constraints.

```c
typedef struct {
    int period;           // How often task runs
    int deadline;         // Must complete by
    int execution_time;   // Max time needed
    int priority;         // Scheduling priority
} RealTimeTask;

bool schedulable(RealTimeTask tasks[], int n) {
    // Schedulability analysis
    // Check if all deadlines can be met
    return true;
}
```

### Algorithms

**Rate Monotonic**:
```python
def rate_monotonic_priority(tasks):
    # Shorter period = higher priority
    return sorted(tasks, key=lambda t: t.period)
```

**Earliest Deadline First**:
```python
def edf_schedule(tasks, current_time):
    # Earliest absolute deadline = highest priority
    return min(tasks, key=lambda t: t.deadline)
```

### Analysis

```
Utilization bound for RM:
U = Σ(Ci/Ti) ≤ n(2^(1/n) - 1)

where:
Ci = execution time of task i
Ti = period of task i
n = number of tasks
```

## Key Takeaways

1. Real-time systems have strict timing requirements
2. Hard real-time must meet all deadlines (safety-critical)
3. Soft real-time tolerates occasional deadline misses
4. Schedulability analysis determines feasibility

## Interview Focus

1. Difference between hard and soft real-time?
2. Explain Rate Monotonic scheduling
3. What is priority inversion? How to prevent?
4. Calculate schedulability for task set
