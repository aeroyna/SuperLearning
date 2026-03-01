# Main Memory

## Overview

Memory management is a critical OS function. Understanding how the OS manages main memory is essential for system design and optimization.

## Topics Covered

1. **[Address Binding](01_address_binding.md)**
   - Compile time binding
   - Load time binding
   - Execution time binding
   - Relocation

2. **[Logical vs Physical Address](02_logical_vs_physical_address.md)**
   - Logical (virtual) address space
   - Physical address space
   - Memory Management Unit (MMU)
   - Address translation
   - Base and limit registers

3. **[Contiguous Memory Allocation](03_contiguous_allocation.md)**
   - Fixed partitioning
   - Variable partitioning
   - First fit, best fit, worst fit
   - Internal vs external fragmentation

4. **[Fragmentation](04_fragmentation.md)**
   - Internal fragmentation
   - External fragmentation
   - Compaction
   - Solutions

5. **[Paging](05_paging.md)**
   - Basic concept
   - Page table
   - Translation Lookaside Buffer (TLB)
   - Hierarchical paging
   - Hashed page tables
   - Inverted page tables
   - Page table structure

6. **[Segmentation](06_segmentation.md)**
   - Segmentation scheme
   - Segment table
   - Segmentation with paging
   - x86 segmentation

## Key Takeaways

- Logical addresses are translated to physical addresses
- Paging eliminates external fragmentation
- TLB speeds up address translation
- Modern systems use paging or paging with segmentation

## Interview Focus

- Explain address translation process
- Calculate page numbers and offsets
- Understand TLB and its importance
- Compare paging vs segmentation
- Solve fragmentation problems
