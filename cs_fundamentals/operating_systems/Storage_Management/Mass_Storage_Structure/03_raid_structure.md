# RAID Structure\n\n## RAID Structure

### Overview

RAID levels 0, 1, 5, 6, 10

### Structure

Physical organization of mass storage devices.

### Algorithms

```c
typedef struct {
    int cylinder;
    int sector;
    int track;
} DiskRequest;

void disk_schedule(DiskRequest requests[], int n) {
    // Scheduling algorithm
    // Minimize seek time and rotational latency
}
```

### Performance

```python
def calculate_seek_time(requests, algorithm):
    total_time = 0
    current_position = 0
    
    for request in sorted(requests, key=algorithm):
        seek_distance = abs(request - current_position)
        total_time += seek_distance
        current_position = request
        
    return total_time
```

### Comparison

Different strategies for different workloads.

## Key Takeaways

1. Disk scheduling minimizes seek time
2. RAID provides redundancy and performance
3. SSDs have different characteristics than HDDs
4. Storage technology impacts OS design

## Interview Focus

1. Explain disk scheduling algorithms
2. Compare RAID levels
3. Calculate seek time for request sequence
4. How do SSDs differ from HDDs?
