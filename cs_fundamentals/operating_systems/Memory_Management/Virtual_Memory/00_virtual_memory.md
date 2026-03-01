# Virtual Memory

## Overview

Virtual memory allows execution of processes that are not completely in memory. This is one of the most important OS concepts and frequently asked in interviews.

## Topics Covered

1. **[Demand Paging](01_demand_paging.md)**
   - Concept of demand paging
   - Page fault handling
   - Effective access time calculation
   - Pure demand paging
   - Lazy swapper
   - Performance implications

2. **[Page Replacement Algorithms](02_page_replacement_algorithms.md)**
   - FIFO (First-In-First-Out)
   - Optimal algorithm
   - LRU (Least Recently Used)
   - LRU approximation algorithms (Second Chance, Clock)
   - LFU (Least Frequently Used)
   - MFU (Most Frequently Used)
   - Belady's anomaly
   - Counting-based algorithms

3. **[Thrashing](03_thrashing.md)**
   - What is thrashing?
   - Causes of thrashing
   - Locality model
   - Working set model
   - Page fault frequency
   - Prevention strategies

4. **[Working Set Model](04_working_set_model.md)**
   - Working set concept
   - Working set window
   - Calculating working set size
   - Prepaging
   - Locality of reference

5. **[Memory-Mapped Files](05_memory_mapped_files.md)**
   - Concept and benefits
   - File I/O using memory mapping
   - Shared memory via memory-mapped files
   - Copy-on-write

## Key Takeaways

- Virtual memory decouples logical from physical memory
- Demand paging loads pages only when needed
- Page replacement is needed when memory is full
- LRU is optimal but hard to implement exactly
- Thrashing degrades performance significantly

## Interview Focus

- Solve page replacement algorithm problems
- Calculate effective access time with page faults
- Explain and identify thrashing
- Compare page replacement algorithms
- Understand working set and locality
