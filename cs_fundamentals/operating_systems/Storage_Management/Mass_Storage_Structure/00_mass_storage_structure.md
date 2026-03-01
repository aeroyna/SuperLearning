# Mass Storage Structure

## Overview

Mass storage devices like hard disks and SSDs are critical components. Understanding their structure and management affects file system performance.

## Topics Covered

1. **[Disk Structure](01_disk_structure.md)**
   - Physical structure (platters, tracks, sectors, cylinders)
   - Logical structure (blocks)
   - Disk addressing
   - Disk formatting
   - Boot block

2. **[Disk Scheduling Algorithms](02_disk_scheduling.md)**
   - FCFS (First-Come, First-Served)
   - SSTF (Shortest Seek Time First)
   - SCAN (Elevator algorithm)
   - C-SCAN (Circular SCAN)
   - LOOK and C-LOOK
   - Performance comparison

3. **[RAID Structure](03_raid_structure.md)**
   - RAID levels (0, 1, 5, 6, 10)
   - Striping, mirroring, parity
   - Reliability vs performance
   - RAID trade-offs
   - Hot spares

4. **[SSD and Flash Storage](04_ssd_flash_storage.md)**
   - Flash memory characteristics
   - Wear leveling
   - Garbage collection
   - TRIM command
   - SSD vs HDD comparison

## Key Takeaways

- Disk scheduling minimizes seek time
- RAID provides redundancy and/or performance
- SSDs have different characteristics than HDDs
- Modern systems increasingly use SSDs

## Interview Focus

- Solve disk scheduling problems (calculate seek time)
- Compare RAID levels
- Understand SSD vs HDD trade-offs
- Explain wear leveling in SSDs
