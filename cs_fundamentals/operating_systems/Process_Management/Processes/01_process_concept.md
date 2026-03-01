# Process Concept

## Definition

A **process** is a program in execution. It is the fundamental unit of work in modern operating systems, representing an active entity (unlike a program, which is a passive entity stored on disk).

## Process vs Program

```
Program (Passive):              Process (Active):
┌──────────────────┐           ┌──────────────────────┐
│  Executable File │           │  Program Counter     │
│  on Disk         │           │  Registers           │
│                  │  Load &   │  Memory (Stack/Heap) │
│  .text (code)    │  Execute  │  Open Files          │
│  .data (data)    │  ───────> │  I/O Status          │
│  .bss (uninit)   │           │  CPU State           │
│                  │           │  PID                 │
└──────────────────┘           └──────────────────────┘

Program: Recipe          Process: Cooking
```

**Key Differences**:
- **Program**: Static code and data on disk
- **Process**: Dynamic instance with allocated resources

**Example**:
```
One program (firefox.exe) can create multiple processes:
- Main browser process (PID 1234)
- Renderer process (PID 1235)
- Extension process (PID 1236)
- Plugin process (PID 1237)
```

## Process Components

### 1. Program Code (Text Section)

**Content**: Compiled executable instructions

```c
// Program code example
int main() {
    int x = 10;      // This code is in text section
    int y = 20;      // after compilation
    return x + y;
}
```

**Characteristics**:
- Read-only (typically)
- Shared among multiple instances
- Loaded from executable file

### 2. Current Activity

**Includes**:
- **Program Counter (PC)**: Address of next instruction
- **CPU Registers**: Current values of general-purpose registers
- **Stack Pointer (SP)**: Top of stack

```
CPU State during execution:
┌────────────────────────────┐
│ PC:  0x00401000            │  ← Next instruction
│ SP:  0x00BFFFF0            │  ← Stack top
│ EAX: 0x00000001            │  ← Register values
│ EBX: 0x00000002            │
│ ECX: 0x00000003            │
└────────────────────────────┘
```

### 3. Stack

**Purpose**: Stores temporary data (function parameters, return addresses, local variables)

```
Stack Layout:
┌─────────────────────┐  ← High Address
│  Function 3 Frame   │
│  - Local vars       │
│  - Return address   │
├─────────────────────┤
│  Function 2 Frame   │
│  - Parameters       │
│  - Local vars       │
├─────────────────────┤
│  Function 1 Frame   │
│  - Parameters       │
├─────────────────────┤
│  main() Frame       │
└─────────────────────┘  ← Low Address (SP points here)
```

**Example (C)**:
```c
#include <stdio.h>

void func2(int x) {
    int local = x * 2;  // On stack
    printf("%d\n", local);
}

void func1(int a) {
    int b = a + 1;      // On stack
    func2(b);
}

int main() {
    int num = 10;       // On stack
    func1(num);
    return 0;
}
```

### 4. Data Section

**Contains**:
- **Initialized data**: Global and static variables with initial values
- **Uninitialized data (BSS)**: Global and static variables without initial values

```c
// Data section example
int global_init = 42;        // Initialized data
static int static_init = 10; // Initialized data

int global_uninit;           // BSS (zero-initialized)
static int static_uninit;    // BSS

int main() {
    // These are on stack, NOT data section
    int local = 5;
    return 0;
}
```

**Memory Layout**:
```
┌──────────────────────┐  ← High Address
│  Stack               │  ← Grows down
│  ↓                   │
│                      │
│  (Free Space)        │
│                      │
│  ↑                   │
│  Heap                │  ← Grows up
├──────────────────────┤
│  BSS (uninit data)   │
├──────────────────────┤
│  Data (init data)    │
├──────────────────────┤
│  Text (code)         │
└──────────────────────┘  ← Low Address (0x00400000)
```

### 5. Heap

**Purpose**: Dynamically allocated memory

**Example (C++)**:
```cpp
#include <iostream>

int main() {
    // Stack allocation
    int stack_var = 10;

    // Heap allocation
    int* heap_var = new int(20);

    // Heap allocation (array)
    int* array = new int[100];

    std::cout << "Stack: " << &stack_var << std::endl;
    std::cout << "Heap: " << heap_var << std::endl;

    // Must free heap memory
    delete heap_var;
    delete[] array;

    return 0;
}
```

**Java Heap Example**:
```java
public class ProcessMemory {
    public static void main(String[] args) {
        // All objects in Java are heap-allocated
        Integer obj1 = new Integer(10);  // Heap
        String str = new String("Hello"); // Heap
        int[] array = new int[100];      // Heap

        // Primitives on stack (if local)
        int local = 5;                   // Stack

        // Garbage collector manages heap automatically
    }
}
```

## Process in Memory

**Complete Process Image**:
```
Virtual Address Space (32-bit example):

0xFFFFFFFF  ┌─────────────────────────┐
            │  Kernel Space           │  ← OS kernel (shared)
            │  (1 GB)                 │
0xC0000000  ├─────────────────────────┤
            │  Stack                  │  ← Function calls, local vars
            │  (Max ~8 MB default)    │    Grows downward
            │  ↓                      │
            │                         │
            │  (Unused)               │
            │                         │
            │  ↑                      │
            │  Heap                   │  ← Dynamic allocation
            │  (Grows upward)         │    malloc/new
            ├─────────────────────────┤
            │  BSS (Uninitialized)    │  ← Global uninit vars
            ├─────────────────────────┤
            │  Data (Initialized)     │  ← Global init vars
            ├─────────────────────────┤
            │  Text (Code)            │  ← Program instructions
0x00400000  └─────────────────────────┘
0x00000000  (Reserved/Invalid)
```

## Process Attributes

### Process ID (PID)

**Definition**: Unique identifier for each process

**Example (C - Linux)**:
```c
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>

int main() {
    pid_t pid = getpid();     // Current process ID
    pid_t ppid = getppid();   // Parent process ID

    printf("My PID: %d\n", pid);
    printf("Parent PID: %d\n", ppid);

    return 0;
}
```

**Python Example**:
```python
import os

print(f"Process ID: {os.getpid()}")
print(f"Parent ID: {os.getppid()}")
```

### Process Priority

**Definition**: Determines process scheduling order

**Example (C)**:
```c
#include <sys/resource.h>
#include <stdio.h>

int main() {
    int priority = getpriority(PRIO_PROCESS, 0);
    printf("Current priority: %d\n", priority);

    // Lower number = higher priority (range: -20 to 19)
    setpriority(PRIO_PROCESS, 0, 10);

    return 0;
}
```

## Process Lifecycle

**State Transitions**:
```
     ┌─────────────┐
     │     NEW     │  ← Process being created
     └──────┬──────┘
            │ admitted
            ↓
     ┌──────────────┐
     │    READY     │  ← Waiting for CPU
     └──────┬───────┘
            │ scheduler dispatch
            ↓
     ┌──────────────┐
     │   RUNNING    │  ← Executing on CPU
     └──────┬───────┘
            │
       ┌────┼─────┐
       │    │     │
  interrupt │    I/O or
   timeout  │   event wait
       │    │     │
       ↓    │     ↓
    ┌──────┴─────────┐
    │    READY       │
    └────────────────┘

                 ┌──────────────┐
                 │   WAITING    │  ← Waiting for I/O
                 │  (BLOCKED)   │
                 └──────┬───────┘
                        │ I/O completion
                        ↓
                 ┌──────────────┐
                 │    READY     │
                 └──────────────┘

     ┌──────────────┐
     │  TERMINATED  │  ← Process finished
     └──────────────┘
```

## Single vs Multi-threaded Process

**Single-threaded**:
```
Process:
┌─────────────────────────────────┐
│  Code                           │
│  Data                           │
│  Files                          │
│  ┌───────────────────────────┐  │
│  │  Single Thread            │  │
│  │  - Registers              │  │
│  │  - Stack                  │  │
│  │  - Program Counter        │  │
│  └───────────────────────────┘  │
└─────────────────────────────────┘
```

**Multi-threaded**:
```
Process:
┌─────────────────────────────────────────┐
│  Code (Shared)                          │
│  Data (Shared)                          │
│  Files (Shared)                         │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐ │
│  │Thread 1 │  │Thread 2 │  │Thread 3 │ │
│  │- Regs   │  │- Regs   │  │- Regs   │ │
│  │- Stack  │  │- Stack  │  │- Stack  │ │
│  │- PC     │  │- PC     │  │- PC     │ │
│  └─────────┘  └─────────┘  └─────────┘ │
└─────────────────────────────────────────┘
```

## Key Takeaways

1. **Process** = program in execution with allocated resources
2. **Process memory** includes text, data, BSS, heap, and stack sections
3. **Process attributes** include PID, priority, state, registers, memory, files
4. **Process lifecycle** involves NEW → READY → RUNNING → WAITING → TERMINATED
5. **Multiple processes** can run from same program (different instances)

## Interview Focus

**Common Questions**:
1. What is the difference between a process and a program?
2. Describe the memory layout of a process
3. What is the difference between stack and heap?
4. Explain the process lifecycle and state transitions
5. What is a Process ID (PID)?

**Coding Questions**:
- Write a program that prints its PID and parent PID
- Explain memory allocation (stack vs heap) with code examples
- Demonstrate the difference between global, static, and local variables

**Real-World Examples**:
- How does Chrome use multiple processes?
- Memory layout of a typical application
- Process states in task manager (Windows) or ps (Linux)
