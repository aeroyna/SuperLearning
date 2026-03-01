# System Calls

## Definition

A **system call** is a programmatic interface that allows user-level programs to request services from the operating system kernel. It serves as the bridge between user mode and kernel mode, enabling controlled access to privileged operations.

## Why System Calls?

### Protection and Security
- User programs run in **user mode** with restricted privileges
- Direct hardware access could:
  - Crash the system
  - Corrupt other processes' data
  - Bypass security mechanisms
- System calls provide a **controlled gateway** to privileged operations

### Abstraction
- Hide hardware complexity
- Provide portable interface
- Maintain backward compatibility

## System Call Mechanism

### The System Call Process

```
User Program (User Mode)
    ↓
Library Function (e.g., printf)
    ↓
System Call Wrapper (e.g., write)
    ↓
Trap/Interrupt (Switch to Kernel Mode)
    ↓
Kernel: System Call Handler
    ↓
Kernel: Execute Requested Service
    ↓
Return to User Mode
    ↓
User Program Continues
```

### Detailed Steps

1. **User program calls library function**: e.g., `printf("Hello\n")`
2. **Library prepares system call**: Sets up parameters, system call number
3. **Trap instruction executed**: Switches CPU from user mode to kernel mode
4. **Kernel dispatcher**: Identifies which system call to execute
5. **System call handler**: Executes the requested operation
6. **Return**: Switches back to user mode, returns result to user program

### Mode Switching Example

```c
// User code
int fd = open("file.txt", O_RDONLY);  // User mode
// ↓ Trap to kernel
// Kernel executes open() handler
// ↓ Return to user
// fd now contains file descriptor
```

**What happens internally**:
1. CPU saves user program state (registers, program counter)
2. CPU switches to kernel mode
3. Kernel validates parameters (`"file.txt"`, `O_RDONLY`)
4. Kernel performs file open operation
5. Kernel returns file descriptor
6. CPU restores user program state
7. CPU switches back to user mode

## Categories of System Calls

### 1. Process Control

#### Process Creation and Termination
- `fork()`: Create a new process (Unix/Linux)
- `exec()`: Replace process image
- `exit()`: Terminate process
- `wait()`: Wait for child process

**Example (C - Linux)**:
```c
#include <unistd.h>
#include <sys/wait.h>

int main() {
    pid_t pid = fork();  // Create child process

    if (pid == 0) {
        // Child process
        printf("Child process (PID: %d)\n", getpid());
        exit(0);
    } else if (pid > 0) {
        // Parent process
        printf("Parent process (PID: %d)\n", getpid());
        wait(NULL);  // Wait for child to finish
    }

    return 0;
}
```

**Example (Python)**:
```python
import os
import sys

pid = os.fork()

if pid == 0:
    # Child process
    print(f"Child process (PID: {os.getpid()})")
    sys.exit(0)
else:
    # Parent process
    print(f"Parent process (PID: {os.getpid()})")
    os.wait()  # Wait for child
```

#### Process Attributes
- `getpid()`: Get process ID
- `getppid()`: Get parent process ID
- `setpriority()`: Set process priority
- `getpriority()`: Get process priority

### 2. File Management

#### File Operations
- `open()`: Open a file
- `read()`: Read from file
- `write()`: Write to file
- `close()`: Close file
- `lseek()`: Reposition read/write offset

**Example (C)**:
```c
#include <fcntl.h>
#include <unistd.h>

int main() {
    int fd = open("data.txt", O_RDONLY);
    if (fd < 0) {
        perror("open failed");
        return 1;
    }

    char buffer[100];
    ssize_t bytes = read(fd, buffer, sizeof(buffer));

    close(fd);
    return 0;
}
```

**Example (Java)**:
```java
import java.io.*;

public class FileExample {
    public static void main(String[] args) {
        try (FileInputStream fis = new FileInputStream("data.txt")) {
            // Internally uses native read() system call
            byte[] buffer = new byte[100];
            int bytesRead = fis.read(buffer);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

#### File Attributes
- `stat()`: Get file status
- `chmod()`: Change file permissions
- `chown()`: Change file owner

### 3. Device Management

- `ioctl()`: Device-specific control operations
- `read()`, `write()`: Also used for devices
- `open()`, `close()`: Also used for devices

**In Unix/Linux, everything is a file**:
```c
// Writing to a file
int fd = open("file.txt", O_WRONLY);
write(fd, "Hello", 5);

// Writing to a device (e.g., terminal)
write(1, "Hello\n", 6);  // 1 is stdout file descriptor
```

### 4. Information Maintenance

- `time()`: Get current time
- `gettimeofday()`: Get time with microsecond precision
- `getrusage()`: Get resource usage
- `uname()`: Get system information

**Example (C)**:
```c
#include <sys/time.h>
#include <sys/resource.h>

int main() {
    struct rusage usage;
    getrusage(RUSAGE_SELF, &usage);

    printf("User CPU time: %ld.%06ld sec\n",
           usage.ru_utime.tv_sec, usage.ru_utime.tv_usec);
    printf("System CPU time: %ld.%06ld sec\n",
           usage.ru_stime.tv_sec, usage.ru_stime.tv_usec);

    return 0;
}
```

### 5. Communication

#### Pipes
- `pipe()`: Create a pipe
- `read()`, `write()`: Communicate via pipe

**Example (C)**:
```c
#include <unistd.h>

int main() {
    int pipefd[2];
    pipe(pipefd);  // Create pipe

    if (fork() == 0) {
        // Child writes to pipe
        close(pipefd[0]);  // Close read end
        write(pipefd[1], "Hello parent", 12);
        close(pipefd[1]);
    } else {
        // Parent reads from pipe
        close(pipefd[1]);  // Close write end
        char buf[20];
        read(pipefd[0], buf, 20);
        close(pipefd[0]);
    }

    return 0;
}
```

#### Shared Memory
- `shmget()`: Get shared memory segment
- `shmat()`: Attach shared memory
- `shmdt()`: Detach shared memory

#### Message Queues
- `msgget()`: Get message queue
- `msgsnd()`: Send message
- `msgrcv()`: Receive message

#### Sockets
- `socket()`: Create socket
- `bind()`: Bind to address
- `listen()`, `accept()`: Server operations
- `connect()`: Client operation
- `send()`, `recv()`: Data transfer

### 6. Protection

- `chmod()`: Change file permissions
- `umask()`: Set default permissions mask
- `chown()`: Change file ownership
- `setuid()`: Set user ID
- `setgid()`: Set group ID

## System Call Interface

### POSIX Standard
- Portable Operating System Interface
- Defines standard system call API
- Ensures portability across Unix-like systems

### System Call Numbers
Each system call has a unique number:
```c
// Linux x86_64 examples
#define __NR_read    0
#define __NR_write   1
#define __NR_open    2
#define __NR_close   3
#define __NR_fork    57
```

### Parameter Passing

#### 1. Registers (Fastest)
```
System call number  → %rax (x86_64)
Parameters          → %rdi, %rsi, %rdx, %r10, %r8, %r9
Return value        ← %rax
```

#### 2. Memory Block
- Parameters stored in memory
- Register points to block
- Used for many parameters

#### 3. Stack
- Parameters pushed onto stack
- Less common in modern systems

## System Call Overhead

### Cost Components
1. **Mode switch**: User → Kernel → User (expensive)
2. **Context save/restore**: Save registers, restore later
3. **Parameter validation**: Check for valid pointers, ranges
4. **Execution**: Actual work performed

### Performance Implications
```
Regular function call:  ~1-5 nanoseconds
System call:           ~100-300 nanoseconds
```

**Optimization strategies**:
- **Buffering**: Batch multiple operations (e.g., `fwrite()` buffers data)
- **Caching**: Keep frequently used data in user space
- **vDSO**: Virtual Dynamic Shared Object (Linux) - some syscalls in user space

## System Calls vs Library Calls

| Aspect | System Call | Library Call |
|--------|-------------|--------------|
| **Mode** | Switches to kernel mode | Stays in user mode |
| **Overhead** | High (mode switch) | Low (function call) |
| **Portability** | OS-specific | More portable |
| **Examples** | `open()`, `read()`, `fork()` | `printf()`, `malloc()`, `strlen()` |

**Example**:
```c
printf("Hello\n");  // Library call
    ↓ (internally calls)
write(1, "Hello\n", 6);  // System call
```

## Common System Calls by OS

### Linux/Unix
- `fork()`, `exec()`, `wait()`
- `open()`, `read()`, `write()`, `close()`
- `pipe()`, `socket()`
- `mmap()`, `brk()`

### Windows
- `CreateProcess()`
- `CreateFile()`, `ReadFile()`, `WriteFile()`
- `CreatePipe()`, `CreateSocket()`
- `VirtualAlloc()`, `HeapAlloc()`

## System Call Wrappers

Most languages provide wrappers:

**C (POSIX)**:
```c
ssize_t read(int fd, void *buf, size_t count);
```

**Python**:
```python
os.read(fd, n)  # Wrapper around read() syscall
```

**Java**:
```java
// FileInputStream.read() eventually calls native read()
```

**JavaScript (Node.js)**:
```javascript
fs.readSync(fd, buffer, offset, length, position);
// Binds to libuv, which calls read() syscall
```

## Debugging System Calls

### strace (Linux)
```bash
strace ls
# Shows all system calls made by 'ls' command

# Output example:
# execve("/bin/ls", ["ls"], ...) = 0
# open("/etc/ld.so.cache", O_RDONLY) = 3
# read(3, "\177ELF...", 832) = 832
```

### dtrace (macOS/BSD)
```bash
dtrace -n 'syscall:::entry { @num[execname] = count(); }'
```

## Interview Focus

**Common Questions**:
1. What is a system call and why is it needed?
2. Explain the steps involved in executing a system call
3. What is the difference between a system call and a library function?
4. Why are system calls expensive?
5. Name the main categories of system calls with examples

**Coding Questions**:
- Implement a program that forks a child process and uses pipes for IPC
- Write code to read a file using system calls (not library functions)
- Explain what happens when you call `printf()`

**Scenario-Based**:
- A program crashes when accessing memory. What system call would the OS use to handle this?
- How does the OS prevent a malicious program from directly accessing disk hardware?

## Key Takeaways

1. System calls provide **controlled access** to kernel services from user mode
2. They involve a **mode switch** (user → kernel → user), which is expensive
3. Five main categories: **Process Control**, **File Management**, **Device Management**, **Information Maintenance**, **Communication**
4. Library functions often **wrap** system calls, providing higher-level interfaces
5. Understanding system calls is crucial for **system programming** and **performance optimization**
