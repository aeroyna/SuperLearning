# What is an Operating System?

## Definition and Core Purpose

An **Operating System (OS)** is a layer of software that sits between the hardware and application programs, acting as a resource manager and abstraction provider. It serves two primary functions:

1. **Resource Manager**: Manages CPU time, memory space, file storage, and I/O devices among competing programs
2. **Extended Machine**: Provides a clean, simplified interface to the hardware, hiding complexity from application developers

## The Layered View of a Computer System

```
┌─────────────────────────────────────┐
│   Application Programs              │  (User Space)
│   (Compilers, Browsers, Games)      │
├─────────────────────────────────────┤
│   Operating System                  │  (Kernel Space)
│   (Process Mgmt, Memory, I/O, FS)   │
├─────────────────────────────────────┤
│   Hardware                          │
│   (CPU, Memory, Disks, I/O)         │
└─────────────────────────────────────┘
```

## Key Responsibilities

### 1. Process Management
- **Process creation and termination**: The OS creates processes, assigns resources, and cleans up when they finish
- **CPU scheduling**: Decides which process gets CPU time and for how long
- **Process synchronization**: Coordinates access to shared resources
- **Deadlock handling**: Prevents and resolves resource deadlocks

### 2. Memory Management
- **Memory allocation**: Assigns memory blocks to processes
- **Virtual memory**: Provides illusion of unlimited memory using disk space
- **Memory protection**: Prevents processes from accessing each other's memory
- **Caching**: Manages cache hierarchies (L1, L2, L3, RAM)

### 3. File System Management
- **File operations**: Create, read, write, delete files
- **Directory structure**: Organizes files hierarchically
- **Access control**: Controls who can access which files
- **File protection**: Ensures data integrity and security

### 4. I/O System Management
- **Device drivers**: Interfaces between hardware and kernel
- **Buffering and caching**: Optimizes I/O performance
- **Interrupt handling**: Responds to hardware events
- **Error detection**: Identifies and handles I/O errors

## OS as a Resource Allocator

The OS must efficiently allocate resources among competing requests:

- **Fairness**: All processes get a fair share of resources
- **Efficiency**: Maximize system throughput and utilization
- **Protection**: Prevent processes from interfering with each other

## OS as a Control Program

The OS acts as a supervisor that:
- **Controls program execution**: Prevents errors and improper use of the computer
- **Enforces security**: Authenticates users and protects system resources
- **Manages I/O devices**: Controls all I/O operations for proper resource utilization

## Kernel vs User Mode

Modern operating systems operate in two distinct modes:

### Kernel Mode (Privileged Mode)
- Full access to all hardware and memory
- Can execute privileged instructions
- OS kernel runs in this mode
- Direct hardware manipulation allowed

### User Mode (Unprivileged Mode)
- Limited access to hardware
- Cannot execute privileged instructions
- Application programs run here
- Must use system calls to request OS services

```
┌──────────────────────────────────┐
│  User Mode                       │
│  Applications request services   │
│         ↓ (System Call)          │
├──────────────────────────────────┤
│  Kernel Mode                     │
│  OS executes privileged ops      │
│         ↓                        │
│  Hardware Access                 │
└──────────────────────────────────┘
```

## Mode Switching Example

When a program needs to read a file:

1. **User mode**: Application calls `fread()` function
2. **Trap/System call**: CPU switches to kernel mode
3. **Kernel mode**: OS handles the file read operation
4. **Return**: CPU switches back to user mode
5. **User mode**: Application continues with data

## Historical Evolution

### 1. Batch Systems (1950s)
- Jobs submitted in batches
- No user interaction
- Sequential execution
- Example: IBM 701

### 2. Multiprogramming Systems (1960s)
- Multiple programs in memory simultaneously
- CPU switches between jobs when one waits for I/O
- Improved CPU utilization
- Example: IBM OS/360

### 3. Time-Sharing Systems (1970s)
- Interactive computing
- Multiple users share CPU through rapid switching
- Gave illusion of dedicated machine
- Example: UNIX, MULTICS

### 4. Personal Computer Systems (1980s-1990s)
- Single-user, single-tasking initially
- Later evolved to multitasking
- GUI introduced
- Example: MS-DOS → Windows, Classic Mac OS

### 5. Distributed Systems (2000s)
- Resources shared across network
- Appears as single coherent system to users
- Example: Google's infrastructure, Kubernetes

### 6. Cloud and Mobile Systems (2010s-Present)
- Virtualization and containerization
- Mobile-optimized OS design
- Energy-efficient scheduling
- Example: Android, iOS, AWS, Azure

## Modern OS Characteristics

### Concurrency
Multiple processes or threads executing concurrently (or pseudo-concurrently on a single CPU)

### Persistence
File systems provide long-term data storage that survives power loss

### Memory Abstraction
Virtual memory gives each process the illusion of having its own dedicated memory space

### Protection and Security
- **Isolation**: Processes cannot interfere with each other
- **Authentication**: Verify user identity
- **Authorization**: Control resource access
- **Auditing**: Track system usage and security events

## Why Study Operating Systems?

1. **Foundation for all software**: Every program runs on an OS
2. **Performance optimization**: Understanding OS internals helps write efficient code
3. **System programming**: Develop drivers, embedded systems, tools
4. **Interview preparation**: Core topic for FAANG+ technical interviews
5. **Debugging**: Diagnose and fix complex system-level issues

## Key Takeaways

- An OS is both a **resource manager** and **extended machine**
- It provides **abstraction** (simplified interface) and **virtualization** (illusion of dedicated resources)
- Modern OSes operate in **dual mode** (kernel vs user) for protection
- Core responsibilities: **process**, **memory**, **file system**, and **I/O management**
- Understanding OS concepts is crucial for **system programming** and **technical interviews**

## Interview Focus

**Common Questions**:
- What is the difference between kernel mode and user mode?
- What are the main functions of an operating system?
- Explain the concept of virtualization in operating systems
- How does the OS manage resources among competing processes?

**Real-World Examples**:
- Linux kernel architecture
- Windows NT kernel design
- macOS XNU (hybrid kernel)
- Android's modified Linux kernel
