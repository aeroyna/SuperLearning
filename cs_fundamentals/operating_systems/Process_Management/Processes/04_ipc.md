# Inter-Process Communication (IPC)

## Overview

IPC allows processes to exchange data and synchronize actions. Since processes run in separate address spaces, they cannot directly access each other's memory and need IPC mechanisms.

## Key Concepts

### IPC Models

**1. Shared Memory**:
- Processes share a region of memory
- Fast (no kernel involvement after setup)
- Requires synchronization
- Same machine only

**2. Message Passing**:
- Processes send messages through kernel
- Slower (kernel involvement)
- No synchronization needed
- Can work across network

### Communication Patterns
- **Direct**: Process names process explicitly
- **Indirect**: Through mailboxes or ports
- **Synchronous**: Blocking send/receive
- **Asynchronous**: Non-blocking operations

## Implementation

```c
// Shared Memory Example
#include <sys/mman.h>
#include <fcntl.h>

int shm_fd = shm_open("/myshm", O_CREAT | O_RDWR, 0666);
ftruncate(shm_fd, 4096);
void *ptr = mmap(0, 4096, PROT_WRITE, MAP_SHARED, shm_fd, 0);
sprintf(ptr, "Hello");

// Message Queue Example
#include <mqueue.h>

mqd_t mq = mq_open("/myqueue", O_CREAT | O_WRONLY, 0644, &attr);
mq_send(mq, "message", 7, 0);
```

## Examples

**Java Shared Memory**:
```java
import java.nio.*;
import java.nio.channels.*;

RandomAccessFile file = new RandomAccessFile("shared.dat", "rw");
MappedByteBuffer buffer = file.getChannel()
    .map(FileChannel.MapMode.READ_WRITE, 0, 4096);
buffer.put("Hello".getBytes());
```

**Python Message Passing**:
```python
from multiprocessing import Queue

queue = Queue()
queue.put("message")
msg = queue.get()
```

## Key Takeaways

1. **Two main models**: Shared memory and message passing
2. **Shared memory** faster but needs synchronization
3. **Message passing** slower but easier (kernel handles sync)
4. **Choice depends on**: Performance vs ease of use
5. **Distributed systems** require message passing

## Interview Focus

**Common Questions**:
1. Compare shared memory vs message passing
2. When would you use each IPC mechanism?
3. Why does shared memory require synchronization?
4. Explain producer-consumer problem
5. How do sockets work for IPC?
