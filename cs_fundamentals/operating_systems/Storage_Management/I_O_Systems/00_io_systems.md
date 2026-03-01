# I/O Systems

## Overview

I/O management is a major component of operating systems. Efficient I/O handling is critical for overall system performance.

## Topics Covered

1. **[I/O Hardware](01_io_hardware.md)**
   - I/O devices (block vs character)
   - Device controllers
   - Memory-mapped I/O vs Port-mapped I/O
   - Polling
   - Interrupts

2. **[Application I/O Interface](02_application_io_interface.md)**
   - Block device interface
   - Character device interface
   - Network device interface
   - System calls for I/O
   - Blocking vs non-blocking I/O

3. **[Kernel I/O Subsystem](03_kernel_io_subsystem.md)**
   - I/O scheduling
   - Buffering
   - Caching
   - Spooling
   - Device reservation
   - Error handling

4. **[I/O Buffering and Caching](04_io_buffering_caching.md)**
   - Why buffering?
   - Single buffering
   - Double buffering
   - Circular buffering
   - Buffer cache
   - Page cache vs buffer cache

5. **[Direct Memory Access (DMA)](05_dma.md)**
   - DMA concept
   - DMA controller
   - DMA transfer process
   - CPU vs DMA
   - Benefits and overhead

## Key Takeaways

- I/O can be synchronous or asynchronous
- Buffering and caching improve I/O performance
- DMA reduces CPU involvement in I/O
- Interrupt-driven I/O is more efficient than polling

## Interview Focus

- Compare polling, interrupts, and DMA
- Explain buffering strategies
- Understand blocking vs non-blocking I/O
- Describe DMA transfer process
