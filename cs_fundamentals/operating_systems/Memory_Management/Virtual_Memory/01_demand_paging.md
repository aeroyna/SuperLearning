# Demand Paging

## Introduction

**Demand Paging** is a memory management technique where pages are loaded into main memory only when they are requested (demanded) by the executing process. This is the core mechanism behind modern Virtual Memory systems.

*   **Lazy Launcher**: Never swaps a page into memory unless it is needed.
*   **Pager**: The swapper that deals with individual pages.

## The Page Fault Mechanism

When a process tries to access a page that is not currently in memory (marked invalid in the page table), a **Page Fault** trap is triggered. The OS handles this as follows:

```mermaid
flowchart TD
    Start([1. Reference Memory]) --> Check{2. In Memory?}
    
    Check -- Yes --> Success([Success: Read/Write])
    Check -- No --> Trap[3. Trap to OS: Page Fault]
    
    Trap --> Save[Save State]
    Save --> Verify{4. Valid Reference?}
    
    Verify -- No --> Abort([Abort Process: Seg Fault])
    Verify -- Yes --> FreeFrame{5. Free Frame?}
    
    FreeFrame -- Yes --> DiskIO
    FreeFrame -- No --> Replacement[Page Replacement Algorithm]
    Replacement --> SwapOut[Swap Out Victim Page]
    SwapOut --> DiskIO
    
    DiskIO[6. Read Page from Disk] --> Wait[Wait for Disk I/O]
    Wait --> Update[7. Update Page Table]
    Update --> Reset[8. Reset Page Table Bit: Valid]
    Reset --> Restart([9. Restart Instruction])
    
    style Trap fill:#ffcdd2
    style DiskIO fill:#fff9c4
    style Restart fill:#c8e6c9
```

## Detailed Steps

1.  **Memory Reference**: The CPU tries to access a logical address.
2.  **Page Table Check**: Hardware looks at the Valid-Invalid bit. 0 = Invalid/Not in RAM.
3.  **Trap**: A hardware interrupt (trap) transfers control to the OS.
4.  **Check Legality**: OS determines if the address was actually valid (part of the process address space) or an illegal access (Segmentation Fault).
5.  **Find Free Frame**: OS looks for an empty physical frame in the `free-frame list`.
    *   If no free frame, run **Page Replacement Algorithm** (select victim, swap out).
6.  **Disk Operation**: Schedule a disk read to bring the missing page into the allocated frame.
7.  **Wait**: The managing process is blocked while disk I/O occurs. Context switch to another process.
8.  **Update Tables**: When disk I/O completes, update the Page Table (set Frame #, Valid bit = 1).
9.  **Restart**: Restore the register state and restart the instruction that caused the trap.

## Advantages & Disadvantages

| Pros | Cons |
| :--- | :--- |
| **Large Virtual Memory**: Programs > Physical RAM | **Latency**: Access time increases on page faults |
| **Multi-programming**: More processes in RAM | **Thrashing**: If Working Set > RAM, excessive swapping |
| **Fast Startup**: Only load needed code | **Complexity**: Hardware support required (TLB, Swap) |

## Performance (Effective Access Time)

The performance depends heavily on the **Page Fault Rate ($p$)**.

$$ \text{EAT} = (1 - p) \times \text{Memory Access} + p \times \text{Page Fault Overhead} $$

*   Memory Access: ~10-100 nanoseconds.
*   Page Fault Overhead: ~5-10 milliseconds (disk seek + latency).

Even a small $p$ (e.g., 0.1%) can degrade performance by factor of 1000!
