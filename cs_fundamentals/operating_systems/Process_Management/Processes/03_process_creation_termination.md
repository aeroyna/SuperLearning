# Process Creation and Termination

## Overview

Process creation and termination are fundamental operations in operating systems. Processes are created through system calls and terminated either normally or abnormally.

## Key Concepts

### Fork-Exec Model (Unix/Linux)

**fork()**: Creates a child process by duplicating the parent.
**exec()**: Replaces the process image with a new program.
**wait()**: Parent waits for child completion.

### Process Creation Steps
1. Allocate PCB
2. Assign unique PID
3. Copy or share parent's resources
4. Initialize PCB
5. Place in ready queue

### Process Termination
- **Normal**: exit() or return from main
- **Abnormal**: Signal (SIGKILL, SIGSEGV)
- **Zombie**: Terminated but not reaped by parent
- **Orphan**: Parent terminated, adopted by init

## Implementation

```c
#include <unistd.h>
#include <sys/wait.h>
#include <stdio.h>

int main() {
    pid_t pid = fork();
    
    if (pid == 0) {
        // Child process
        printf("Child PID: %d\n", getpid());
        execl("/bin/ls", "ls", "-l", NULL);
    } else {
        // Parent process
        printf("Parent PID: %d, Child PID: %d\n", getpid(), pid);
        wait(NULL);  // Wait for child
    }
    
    return 0;
}
```

## Examples

**Python Process Creation**:
```python
import os
import sys

pid = os.fork()

if pid == 0:
    print(f"Child: PID={os.getpid()}")
    sys.exit(0)
else:
    print(f"Parent: PID={os.getpid()}, Child={pid}")
    os.wait()
```

**Windows CreateProcess**:
```cpp
#include <windows.h>

STARTUPINFO si;
PROCESS_INFORMATION pi;
CreateProcess("app.exe", NULL, NULL, NULL, FALSE, 0, NULL, NULL, &si, &pi);
WaitForSingleObject(pi.hProcess, INFINITE);
CloseHandle(pi.hProcess);
```

## Key Takeaways

1. **fork()** creates child by duplicating parent
2. **exec()** replaces process image
3. **Zombie** processes occur when parent doesn't call wait()
4. **Orphan** processes are adopted by init (PID 1)
5. **Windows** uses CreateProcess() instead of fork-exec

## Interview Focus

**Common Questions**:
1. Explain fork() system call
2. What is the difference between fork() and exec()?
3. What are zombie and orphan processes?
4. How does Windows create processes?
5. What happens if parent doesn't wait()?
