# Storage Structure

## Overview

Storage devices in a computer system form a hierarchy based on **speed**, **cost**, and **capacity**. Understanding this hierarchy is crucial for optimizing system performance and managing resources effectively.

## Storage Hierarchy

```
                    Speed (Fast)
                    Cost (High)
                    Capacity (Small)
                         ↑
    ┌────────────────────────────────────┐
    │    CPU Registers                   │  < 1 ns
    ├────────────────────────────────────┤
    │    L1 Cache                        │  ~1 ns
    ├────────────────────────────────────┤
    │    L2 Cache                        │  ~4 ns
    ├────────────────────────────────────┤
    │    L3 Cache                        │  ~10 ns
    ├────────────────────────────────────┤
    │    Main Memory (RAM)               │  ~100 ns
    ├────────────────────────────────────┤
    │    SSD (Solid State Drive)         │  ~100 μs
    ├────────────────────────────────────┤
    │    HDD (Hard Disk Drive)           │  ~10 ms
    ├────────────────────────────────────┤
    │    Optical Disk, Magnetic Tape     │  ~seconds
    └────────────────────────────────────┘
                         ↓
                    Speed (Slow)
                    Cost (Low)
                    Capacity (Large)
```

## Primary Storage (Volatile)

### CPU Registers

**Characteristics**:
- **Fastest** storage available
- **Smallest** capacity (typically 32-64 registers)
- **Volatile**: Data lost when power is off
- **Built into CPU**

**Types**:
- **General-purpose registers**: For arithmetic and data manipulation
- **Special-purpose registers**: Program Counter (PC), Stack Pointer (SP), Status Register

**Access Time**: Less than 1 nanosecond

**Example (x86_64 Architecture)**:
```
General Purpose:
RAX, RBX, RCX, RDX, RSI, RDI, RBP, RSP, R8-R15

Special Purpose:
RIP (Instruction Pointer)
RFLAGS (Status flags)
```

### Cache Memory

Modern CPUs use a hierarchy of cache levels to bridge the speed gap between CPU and main memory.

#### L1 Cache (Level 1)
- **Location**: On CPU die
- **Size**: 32-64 KB per core
- **Speed**: ~1 ns access time
- **Split**: Separate instruction cache (I-cache) and data cache (D-cache)

#### L2 Cache (Level 2)
- **Location**: On CPU die
- **Size**: 256-512 KB per core
- **Speed**: ~4 ns access time
- **Unified**: Stores both instructions and data

#### L3 Cache (Level 3)
- **Location**: On CPU die (shared)
- **Size**: 8-32 MB (shared across all cores)
- **Speed**: ~10 ns access time
- **Shared**: Among all cores in the CPU

**Cache Organization**:
```
┌──────────────────────────────────┐
│         CPU Core 0               │
│  ┌──────────┐  ┌──────────┐     │
│  │ L1-I     │  │ L1-D     │     │
│  │ 32 KB    │  │ 32 KB    │     │
│  └──────────┘  └──────────┘     │
│         │           │            │
│         └─────┬─────┘            │
│               │                  │
│         ┌─────▼──────┐           │
│         │  L2 Cache  │           │
│         │   256 KB   │           │
│         └─────┬──────┘           │
└───────────────┼──────────────────┘
                │
        ┌───────▼───────┐
        │   L3 Cache    │
        │   16 MB       │
        │   (Shared)    │
        └───────┬───────┘
                │
        ┌───────▼───────┐
        │  Main Memory  │
        │   16 GB RAM   │
        └───────────────┘
```

#### Cache Line
- **Definition**: Smallest unit of data transfer between cache and memory
- **Typical size**: 64 bytes
- **Principle**: Spatial locality - if byte N is accessed, bytes N+1, N+2, ... are likely to be accessed soon

**Example (C)**:
```c
// Cache-friendly code (sequential access)
int sum = 0;
for (int i = 0; i < 1000; i++) {
    sum += array[i];  // Sequential access, good cache locality
}

// Cache-unfriendly code (stride access)
int sum = 0;
for (int i = 0; i < 1000; i += 16) {
    sum += array[i];  // Large stride, poor cache locality
}
```

### Main Memory (RAM)

**Characteristics**:
- **Volatile**: Loses data when power is off
- **Random access**: Any location accessible in constant time
- **Typical size**: 8-64 GB in modern systems
- **Access time**: ~100 ns

**Types**:
- **DRAM (Dynamic RAM)**: Needs periodic refresh, cheaper, slower
- **SRAM (Static RAM)**: No refresh needed, faster, used for cache

**Memory Organization**:
```
Main Memory (Physical)
┌─────────────────────┐  0x00000000
│   Kernel Space      │
│   (OS Code & Data)  │
├─────────────────────┤  0x80000000
│                     │
│   User Space        │
│   (Applications)    │
│                     │
├─────────────────────┤
│   Memory Mapped I/O │
├─────────────────────┤
│   Free Space        │
└─────────────────────┘  0xFFFFFFFF
```

**Memory Access Example (C)**:
```c
#include <stdio.h>
#include <stdlib.h>

int main() {
    // Stack allocation (fast, automatic cleanup)
    int stack_var = 42;

    // Heap allocation (slower, manual management)
    int *heap_var = (int*)malloc(sizeof(int));
    *heap_var = 42;

    printf("Stack: %p, Heap: %p\n", &stack_var, heap_var);

    free(heap_var);
    return 0;
}
```

## Secondary Storage (Non-Volatile)

### Hard Disk Drive (HDD)

**Characteristics**:
- **Non-volatile**: Retains data without power
- **Mechanical**: Rotating platters and moving read/write heads
- **Capacity**: 1-20 TB
- **Access time**: 5-15 ms
- **Sequential read**: 100-200 MB/s
- **Random I/O**: 100-200 IOPS

**Physical Structure**:
```
┌─────────────────────────────────┐
│   Platter (rotating disk)       │
│                                 │
│   ┌───────────────────┐         │
│   │     Track         │         │
│   │   ┌─────────┐     │         │
│   │   │ Sector  │     │         │
│   │   └─────────┘     │         │
│   └───────────────────┘         │
│         ↑                       │
│    Read/Write Head              │
│    (Moves radially)             │
└─────────────────────────────────┘
```

**Components**:
- **Platter**: Magnetic disk that stores data
- **Track**: Concentric circles on platter
- **Sector**: Smallest addressable unit (typically 512 bytes or 4 KB)
- **Cylinder**: All tracks at same position on different platters
- **Head**: Reads/writes magnetic data

**Access Time Calculation**:
```
Total Access Time = Seek Time + Rotational Latency + Transfer Time

Where:
- Seek Time: Time to move head to correct track (3-12 ms)
- Rotational Latency: Time for sector to rotate under head (avg 4-8 ms)
- Transfer Time: Time to read/write data (depends on size)
```

### Solid State Drive (SSD)

**Characteristics**:
- **Non-volatile**: Flash memory
- **No moving parts**: Electronic storage
- **Capacity**: 256 GB - 4 TB
- **Access time**: 50-100 μs
- **Sequential read**: 500-7000 MB/s
- **Random I/O**: 10,000-100,000 IOPS

**Structure**:
```
SSD Controller
├── NAND Flash Memory
│   ├── Block 0
│   │   ├── Page 0 (4 KB)
│   │   ├── Page 1 (4 KB)
│   │   └── ... (256 pages)
│   ├── Block 1
│   └── ...
├── DRAM Cache
└── Controller Firmware
```

**NAND Flash Characteristics**:
- **Page**: Smallest read/write unit (4-16 KB)
- **Block**: Smallest erase unit (256-512 pages)
- **Write limitation**: Must erase entire block before rewriting
- **Wear leveling**: Distributes writes evenly to extend lifespan

**SSD vs HDD Comparison**:
```c++
// Performance simulation (conceptual)
class StorageDevice {
public:
    virtual long accessTime() = 0;
    virtual long readSpeed() = 0;
};

class HDD : public StorageDevice {
public:
    long accessTime() override {
        return 10000; // 10 ms in microseconds
    }

    long readSpeed() override {
        return 150; // MB/s
    }
};

class SSD : public StorageDevice {
public:
    long accessTime() override {
        return 100; // 100 μs
    }

    long readSpeed() override {
        return 3500; // MB/s
    }
};
```

### NVMe (Non-Volatile Memory Express)

**Characteristics**:
- **Interface**: PCIe instead of SATA
- **Parallelism**: Up to 64K queues with 64K commands each
- **Latency**: 10-20 μs
- **Speed**: Up to 7000 MB/s

**Comparison**:
```
SATA SSD:
- Interface: SATA III (6 Gbps)
- Max Speed: ~600 MB/s
- Queue Depth: 32

NVMe SSD:
- Interface: PCIe 4.0 (16 GT/s per lane)
- Max Speed: ~7000 MB/s (4 lanes)
- Queue Depth: 64K
```

## Storage Management

### Caching

**Principle**: Keep frequently accessed data in faster storage.

**Cache Hit vs Miss**:
```
Cache Hit:  Data found in cache → Fast access
Cache Miss: Data not in cache → Must fetch from slower storage
```

**Performance Impact**:
```
Effective Access Time = (Hit Rate × Cache Access Time) +
                        (Miss Rate × Memory Access Time)

Example:
- Cache hit rate: 95%
- Cache access time: 1 ns
- Memory access time: 100 ns

EAT = (0.95 × 1) + (0.05 × 100) = 0.95 + 5 = 5.95 ns
```

**Cache Replacement Policies**:
- **LRU (Least Recently Used)**: Replace least recently accessed item
- **LFU (Least Frequently Used)**: Replace least frequently accessed item
- **FIFO (First In First Out)**: Replace oldest item
- **Random**: Replace random item

**Example (Python - LRU Cache)**:
```python
from functools import lru_cache

@lru_cache(maxsize=128)
def expensive_operation(n):
    # Simulates expensive computation
    result = 0
    for i in range(n):
        result += i
    return result

# First call: Cache miss, computes result
print(expensive_operation(1000000))

# Second call: Cache hit, returns cached result
print(expensive_operation(1000000))  # Much faster
```

### Buffering

**Definition**: Temporary storage area for data being transferred between two devices.

**Purpose**:
- **Speed matching**: Bridge speed gap between fast and slow devices
- **Batch processing**: Accumulate small writes into larger ones
- **Reduce system calls**: Minimize expensive kernel transitions

**Example (C - Buffered I/O)**:
```c
#include <stdio.h>

int main() {
    FILE *fp = fopen("output.txt", "w");

    // Buffered writes (fast)
    for (int i = 0; i < 1000; i++) {
        fprintf(fp, "Line %d\n", i);  // Buffered
    }

    fflush(fp);  // Force buffer to disk
    fclose(fp);

    return 0;
}
```

**Java Buffering Example**:
```java
import java.io.*;

public class BufferedExample {
    public static void main(String[] args) throws IOException {
        // Unbuffered (slow)
        FileWriter fw = new FileWriter("output.txt");
        for (int i = 0; i < 1000; i++) {
            fw.write("Line " + i + "\n");  // Each write is a system call
        }
        fw.close();

        // Buffered (fast)
        BufferedWriter bw = new BufferedWriter(new FileWriter("output.txt"));
        for (int i = 0; i < 1000; i++) {
            bw.write("Line " + i + "\n");  // Writes to buffer
        }
        bw.close();  // Flushes buffer to disk
    }
}
```

## Storage Access Patterns

### Sequential Access
- **Access pattern**: Read/write data in order
- **Performance**: Very fast on both HDD and SSD
- **Use case**: Log files, video streaming

### Random Access
- **Access pattern**: Read/write data in non-sequential order
- **Performance**: Slow on HDD, fast on SSD
- **Use case**: Database queries, file system operations

**Performance Comparison**:
```python
import time
import random

def sequential_access(data):
    """Simulates sequential access pattern"""
    start = time.time()
    result = 0
    for i in range(len(data)):
        result += data[i]
    return time.time() - start

def random_access(data):
    """Simulates random access pattern"""
    start = time.time()
    result = 0
    indices = list(range(len(data)))
    random.shuffle(indices)
    for i in indices:
        result += data[i]
    return time.time() - start

# Test with large array
data = [i for i in range(10000000)]

print(f"Sequential: {sequential_access(data):.4f}s")
print(f"Random: {random_access(data):.4f}s")
```

## Memory-Mapped I/O

**Definition**: Map file contents directly into virtual memory address space.

**Advantages**:
- **Fast access**: No system calls for reading/writing
- **Shared memory**: Multiple processes can access same file
- **Lazy loading**: Pages loaded on demand

**Example (C - mmap)**:
```c
#include <sys/mman.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/stat.h>

int main() {
    int fd = open("data.txt", O_RDWR);
    struct stat sb;
    fstat(fd, &sb);

    // Map file to memory
    char *mapped = mmap(NULL, sb.st_size, PROT_READ | PROT_WRITE,
                        MAP_SHARED, fd, 0);

    // Access file like memory
    mapped[0] = 'X';  // Modifies file

    // Unmap and close
    munmap(mapped, sb.st_size);
    close(fd);

    return 0;
}
```

**Java Memory-Mapped Files**:
```java
import java.io.*;
import java.nio.*;
import java.nio.channels.*;

public class MemoryMappedFile {
    public static void main(String[] args) throws IOException {
        RandomAccessFile file = new RandomAccessFile("data.txt", "rw");
        FileChannel channel = file.getChannel();

        // Map file to memory
        MappedByteBuffer buffer = channel.map(
            FileChannel.MapMode.READ_WRITE, 0, channel.size()
        );

        // Access file like memory
        buffer.put(0, (byte)'X');

        channel.close();
        file.close();
    }
}
```

## Key Takeaways

1. **Storage hierarchy** balances speed, cost, and capacity from registers to magnetic tape
2. **Cache memory** (L1/L2/L3) bridges the speed gap between CPU and main memory
3. **SSDs** are 100x faster than HDDs for random access, crucial for modern systems
4. **Caching** and **buffering** are key techniques for optimizing I/O performance
5. **Access patterns** (sequential vs random) dramatically affect performance
6. **Memory-mapped I/O** provides fast file access by mapping files to virtual memory

## Interview Focus

**Common Questions**:
1. Explain the storage hierarchy and why it exists
2. What is the difference between L1, L2, and L3 cache?
3. Compare HDD vs SSD in terms of performance and architecture
4. What is cache locality and why does it matter?
5. Explain memory-mapped I/O and its advantages

**Coding Questions**:
- Write code demonstrating cache-friendly vs cache-unfriendly array access
- Implement a simple LRU cache
- Compare buffered vs unbuffered I/O performance

**Real-World Examples**:
- Why database systems prefer SSDs over HDDs
- How operating systems use page cache to improve file I/O
- Impact of cache line size on parallel programming
