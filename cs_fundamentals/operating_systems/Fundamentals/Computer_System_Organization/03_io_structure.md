# I/O Structure

## Overview

Input/Output (I/O) systems manage communication between the computer and external devices. Efficient I/O is critical for system performance, as I/O operations are typically much slower than CPU and memory operations.

## I/O Hardware Components

### Device Controller

**Definition**: Hardware that controls one or more I/O devices and acts as an interface between the device and the system bus.

**Components**:
```
┌──────────────────────────────────────┐
│     Device Controller                │
├──────────────────────────────────────┤
│  1. Control Register                 │  ← CPU sets commands
│  2. Status Register                  │  ← CPU reads status
│  3. Data Register                    │  ← Data transfer
│  4. Local Buffer                     │  ← Temporary storage
└──────────────────────────────────────┘
         ↕
    I/O Device
```

**Example Controllers**:
- **Disk controller**: Manages hard drives
- **USB controller**: Manages USB devices
- **Graphics controller**: Manages display
- **Network interface controller (NIC)**: Manages network communication

### Device Driver

**Definition**: Software module that provides interface between OS and device controller.

**Responsibilities**:
- Translate OS commands to device-specific instructions
- Handle device interrupts
- Manage device buffers
- Report errors to OS

**Layered Architecture**:
```
┌─────────────────────────────┐
│  Application                │
├─────────────────────────────┤
│  System Call Interface      │
├─────────────────────────────┤
│  Kernel I/O Subsystem       │
├─────────────────────────────┤
│  Device Driver              │  ← OS-specific
├─────────────────────────────┤
│  Device Controller          │  ← Hardware
├─────────────────────────────┤
│  I/O Device                 │
└─────────────────────────────┘
```

## I/O Methods

### 1. Programmed I/O (Polling)

**Mechanism**: CPU repeatedly checks device status until operation completes.

**Process**:
1. CPU initiates I/O operation
2. CPU continuously polls status register
3. When ready, CPU transfers data
4. CPU continues normal operation

**Advantages**:
- Simple to implement
- No additional hardware required

**Disadvantages**:
- **Busy waiting**: CPU wastes cycles polling
- Inefficient for slow devices
- Cannot do other work while waiting

**Example (C - Polling)**:
```c
#include <stdio.h>

// Simulated device registers
volatile unsigned int *status_register = (unsigned int *)0x3F000000;
volatile unsigned int *data_register = (unsigned int *)0x3F000004;

#define DEVICE_READY 0x01
#define DEVICE_BUSY  0x00

void programmed_io_read(char *buffer, int size) {
    for (int i = 0; i < size; i++) {
        // Busy waiting - polls until device ready
        while (*status_register == DEVICE_BUSY) {
            // CPU is stuck here, wasting cycles
        }

        // Transfer one byte
        buffer[i] = *data_register;
    }
}

int main() {
    char buffer[1024];
    programmed_io_read(buffer, 1024);
    return 0;
}
```

**Performance**:
```
CPU Utilization during I/O: ~100% (all busy waiting)
Throughput: Limited by polling overhead
```

### 2. Interrupt-Driven I/O

**Mechanism**: Device sends interrupt when ready, allowing CPU to do other work.

**Process**:
```
1. CPU initiates I/O operation
2. CPU continues with other tasks
3. Device completes operation
4. Device sends interrupt signal
5. CPU handles interrupt
6. CPU resumes interrupted task
```

**Interrupt Handling**:
```
┌────────────────────────────────────┐
│  CPU executing Process A           │
│         ↓                          │
│  I/O request issued                │
│         ↓                          │
│  CPU executes Process B            │
│         ↓                          │
│  ← Interrupt arrives               │
│         ↓                          │
│  Save Process B state              │
│         ↓                          │
│  Execute Interrupt Handler (ISR)   │
│         ↓                          │
│  Restore Process B state           │
│         ↓                          │
│  Continue Process B                │
└────────────────────────────────────┘
```

**Example (C - Conceptual)**:
```c
#include <signal.h>
#include <stdio.h>

// Interrupt Service Routine (ISR)
void io_interrupt_handler(int signum) {
    // Save current context (done by hardware/OS)

    // Handle the interrupt
    char data = read_device_register();
    process_data(data);

    // Signal completion
    io_complete_flag = 1;

    // Return (restore context)
}

void interrupt_driven_io() {
    // Register interrupt handler
    signal(SIGIO, io_interrupt_handler);

    // Initiate I/O operation
    start_device_read();

    // CPU can do other work here
    while (!io_complete_flag) {
        do_useful_work();  // Not busy waiting!
    }
}
```

**Advantages**:
- CPU free to do other work
- More efficient than polling
- Responsive to device

**Disadvantages**:
- Context switching overhead
- Interrupt handling overhead
- Complex to implement

**Performance**:
```
CPU Utilization: ~80-90% (can do useful work)
Throughput: Much higher than polling
```

### 3. Direct Memory Access (DMA)

**Mechanism**: Dedicated hardware controller transfers data directly between device and memory without CPU intervention.

**DMA Architecture**:
```
┌──────────┐         ┌──────────────┐
│   CPU    │◄───────►│    Memory    │
└──────────┘         └──────────────┘
     │                      ▲
     │                      │ Direct
     │                      │ Transfer
     ▼                      │
┌──────────────┐            │
│ DMA Controller├────────────┘
└──────────────┘
     │
     ▼
┌──────────────┐
│  I/O Device  │
└──────────────┘
```

**DMA Transfer Process**:
1. CPU programs DMA controller:
   - Source/destination address
   - Transfer size
   - Direction (read/write)
2. DMA controller takes over bus
3. DMA transfers data directly to/from memory
4. DMA sends interrupt when complete
5. CPU processes completion

**Example (C - DMA Programming)**:
```c
#include <stdio.h>

typedef struct {
    unsigned int source_addr;
    unsigned int dest_addr;
    unsigned int transfer_size;
    unsigned int control;
} DMA_Descriptor;

#define DMA_CONTROL_START   0x01
#define DMA_CONTROL_READ    0x02
#define DMA_CONTROL_WRITE   0x04

void dma_transfer(void *dest, void *src, size_t size) {
    DMA_Descriptor *dma = (DMA_Descriptor *)0x40000000;

    // Program DMA controller
    dma->source_addr = (unsigned int)src;
    dma->dest_addr = (unsigned int)dest;
    dma->transfer_size = size;
    dma->control = DMA_CONTROL_START | DMA_CONTROL_READ;

    // DMA now transfers data in background
    // CPU is free to do other work

    // Wait for DMA completion (interrupt)
    wait_for_dma_interrupt();
}

int main() {
    char disk_buffer[4096];
    char memory_buffer[4096];

    // Transfer 4KB from disk to memory using DMA
    dma_transfer(memory_buffer, disk_buffer, 4096);

    return 0;
}
```

**DMA Modes**:

**1. Burst Mode**:
```
DMA takes control of bus for entire transfer
- Fastest transfer
- CPU blocked during transfer
```

**2. Cycle Stealing**:
```
DMA transfers one byte/word at a time
- CPU and DMA share bus
- Slower transfer, but CPU can still work
```

**3. Transparent Mode**:
```
DMA transfers only when CPU not using bus
- Slowest transfer
- No impact on CPU performance
```

**Advantages**:
- **Minimal CPU involvement**: Only setup and completion
- **High throughput**: Direct memory transfer
- **Efficient**: CPU free for computation

**Disadvantages**:
- **Additional hardware cost**: DMA controller needed
- **Bus contention**: DMA competes with CPU for bus
- **Complex programming**: More setup required

**Performance Comparison**:
```python
# Simulated performance comparison
import time

class IOMethod:
    def __init__(self, name):
        self.name = name

    def transfer(self, size):
        pass

class ProgrammedIO(IOMethod):
    def transfer(self, size):
        # CPU must handle every byte
        cpu_cycles = size * 100  # 100 cycles per byte
        return cpu_cycles

class InterruptDrivenIO(IOMethod):
    def transfer(self, size):
        # Interrupt per block (assume 512 byte blocks)
        blocks = size // 512
        cpu_cycles = blocks * 1000  # 1000 cycles per interrupt
        return cpu_cycles

class DMA(IOMethod):
    def transfer(self, size):
        # Only setup and completion
        cpu_cycles = 200  # Setup overhead
        return cpu_cycles

# Transfer 1 MB
size = 1024 * 1024

pio = ProgrammedIO("Programmed I/O")
interrupt = InterruptDrivenIO("Interrupt-Driven")
dma = DMA("DMA")

print(f"Programmed I/O: {pio.transfer(size):,} CPU cycles")
print(f"Interrupt I/O: {interrupt.transfer(size):,} CPU cycles")
print(f"DMA: {dma.transfer(size):,} CPU cycles")
```

**Output**:
```
Programmed I/O: 104,857,600 CPU cycles
Interrupt I/O: 2,048,000 CPU cycles
DMA: 200 CPU cycles
```

## I/O Buffering Strategies

### 1. No Buffering (Unbuffered I/O)

**Characteristics**:
- Direct transfer between application and device
- Each I/O operation is a system call
- Slowest method

**Example (C)**:
```c
int fd = open("file.txt", O_WRONLY | O_DIRECT);
for (int i = 0; i < 1000; i++) {
    write(fd, &data[i], 1);  // 1000 system calls!
}
close(fd);
```

### 2. Single Buffering

**Mechanism**:
```
Application → OS Buffer → Device
```

**Process**:
1. Application writes to OS buffer
2. OS transfers buffer to device in background
3. Application can continue (if buffer not full)

**Example (C)**:
```c
#include <stdio.h>

FILE *fp = fopen("output.txt", "w");
setvbuf(fp, NULL, _IOFBF, 4096);  // 4KB buffer

for (int i = 0; i < 1000; i++) {
    fprintf(fp, "Line %d\n", i);  // Buffered
}

fclose(fp);  // Flushes buffer
```

### 3. Double Buffering

**Mechanism**:
```
Application → Buffer 1 ⟺ Buffer 2 → Device
               (swap)
```

**Process**:
1. Application writes to Buffer 1
2. While Buffer 1 is being transferred to device, application writes to Buffer 2
3. Buffers swap roles when full

**Advantages**:
- Overlaps computation and I/O
- Smooths out performance variations
- Used in video streaming, audio playback

**Example (C++ - Producer-Consumer)**:
```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>

class DoubleBuffer {
private:
    char buffer1[4096];
    char buffer2[4096];
    char *write_buffer;
    char *read_buffer;
    std::mutex mtx;
    std::condition_variable cv;
    bool ready = false;

public:
    DoubleBuffer() {
        write_buffer = buffer1;
        read_buffer = buffer2;
    }

    void produce(const char *data, size_t size) {
        std::unique_lock<std::mutex> lock(mtx);

        // Write to current write buffer
        memcpy(write_buffer, data, size);

        // Swap buffers
        std::swap(write_buffer, read_buffer);
        ready = true;

        cv.notify_one();
    }

    void consume() {
        std::unique_lock<std::mutex> lock(mtx);
        cv.wait(lock, [this] { return ready; });

        // Process read buffer
        write_to_device(read_buffer, 4096);

        ready = false;
    }
};
```

### 4. Circular Buffering (Ring Buffer)

**Mechanism**:
```
     ┌───┬───┬───┬───┬───┬───┬───┬───┐
     │ 0 │ 1 │ 2 │ 3 │ 4 │ 5 │ 6 │ 7 │
     └───┴───┴───┴───┴───┴───┴───┴───┘
       ▲                       ▲
       │                       │
    Read Ptr              Write Ptr
```

**Usage**: Audio/video streaming, network buffers, logging

**Example (Java)**:
```java
public class CircularBuffer {
    private byte[] buffer;
    private int readPos;
    private int writePos;
    private int size;
    private int capacity;

    public CircularBuffer(int capacity) {
        this.buffer = new byte[capacity];
        this.capacity = capacity;
        this.readPos = 0;
        this.writePos = 0;
        this.size = 0;
    }

    public synchronized void write(byte data) {
        if (size == capacity) {
            throw new RuntimeException("Buffer full");
        }

        buffer[writePos] = data;
        writePos = (writePos + 1) % capacity;
        size++;
    }

    public synchronized byte read() {
        if (size == 0) {
            throw new RuntimeException("Buffer empty");
        }

        byte data = buffer[readPos];
        readPos = (readPos + 1) % capacity;
        size--;
        return data;
    }

    public synchronized int available() {
        return size;
    }
}
```

## I/O Scheduling

### Purpose
Optimize I/O requests to minimize seek time and maximize throughput.

### Algorithms
- **FCFS (First-Come-First-Served)**: Process requests in order
- **SSTF (Shortest Seek Time First)**: Service closest request first
- **SCAN**: Move head in one direction, service requests
- **C-SCAN**: Like SCAN, but only in one direction

*Detailed coverage in Storage Management section*

## I/O Protection

### Privileged Instructions

**Problem**: User programs should not directly access I/O devices.

**Solution**: All I/O instructions are privileged (kernel mode only).

```c
// User mode - cannot do this
out(PORT, data);  // Error: privileged instruction

// Must use system call
write(fd, buffer, size);  // Traps to kernel
    ↓
// Kernel mode - can access hardware
out(PORT, data);  // OK
```

### Memory-Mapped I/O Protection

**Problem**: Memory-mapped device registers could be accessed directly.

**Solution**: OS uses page protection to prevent user-mode access.

```
Device Registers at 0x40000000

Page Table Entry:
┌──────────────────────────────────┐
│  Physical Addr: 0x40000000       │
│  Permissions: Kernel Only        │  ← User access → Page Fault
│  Present: Yes                    │
└──────────────────────────────────┘
```

## Asynchronous I/O

### Synchronous I/O
```
Application: read(fd, buffer, size);
           ↓ (blocks)
         [wait]
           ↓ (data ready)
         return
```

### Asynchronous I/O
```
Application: aio_read(fd, buffer, size);
           ↓ (continues immediately)
         do_other_work();
           ↓
         aio_wait();  // Wait for completion if needed
```

**Example (C - POSIX AIO)**:
```c
#include <aio.h>
#include <fcntl.h>

int main() {
    int fd = open("data.txt", O_RDONLY);
    char buffer[4096];

    struct aiocb cb;
    memset(&cb, 0, sizeof(struct aiocb));
    cb.aio_fildes = fd;
    cb.aio_buf = buffer;
    cb.aio_nbytes = 4096;
    cb.aio_offset = 0;

    // Start asynchronous read
    aio_read(&cb);

    // Do other work while I/O in progress
    do_computation();

    // Wait for I/O completion
    while (aio_error(&cb) == EINPROGRESS) {
        // Can do more work here
    }

    // Check result
    ssize_t bytes = aio_return(&cb);

    close(fd);
    return 0;
}
```

**JavaScript Example (Node.js)**:
```javascript
const fs = require('fs').promises;

async function asyncRead() {
    // Asynchronous file read
    const data = await fs.readFile('data.txt');

    // This runs after read completes
    console.log(data);
}

// Non-blocking
asyncRead();

// This runs immediately, not waiting for read
console.log('Reading file...');
```

## Key Takeaways

1. **Three main I/O methods**: Programmed I/O (polling), interrupt-driven, and DMA
2. **DMA is most efficient** for large transfers, minimizing CPU involvement
3. **Buffering** (single, double, circular) improves I/O performance by overlapping I/O and computation
4. **Interrupt-driven I/O** allows CPU to do useful work instead of busy waiting
5. **I/O protection** ensures user programs cannot directly access hardware
6. **Asynchronous I/O** enables applications to continue while I/O is in progress

## Interview Focus

**Common Questions**:
1. Compare programmed I/O, interrupt-driven I/O, and DMA
2. How does DMA work and what are its advantages?
3. Explain double buffering and its use cases
4. What is the difference between synchronous and asynchronous I/O?
5. Why are I/O instructions privileged?

**Coding Questions**:
- Implement a circular buffer in your favorite language
- Write code demonstrating asynchronous I/O
- Explain interrupt handling at a low level

**Real-World Examples**:
- How video streaming uses double buffering
- Why SSDs don't need disk scheduling algorithms
- How operating systems prevent direct hardware access
