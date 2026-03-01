# IPC Mechanisms: Pipes, Message Queues, Shared Memory

## Overview

This section covers the three primary IPC mechanisms: pipes, message queues, and shared memory. Each has different characteristics, performance, and use cases.

## Key Concepts

### 1. Pipes

**Characteristics**:
- Unidirectional communication channel
- Data flows in one direction
- First-In-First-Out (FIFO)
- Buffer size limited (typically 64KB)

**Types**:
- **Anonymous pipes**: Parent-child communication
- **Named pipes (FIFOs)**: Unrelated processes

### 2. Message Queues

**Characteristics**:
- Messages have priority and type
- Messages remain until read
- Multiple readers/writers
- Kernel manages queue

### 3. Shared Memory

**Characteristics**:
- Fastest IPC mechanism
- Direct memory access
- Requires synchronization (semaphores, mutexes)
- No kernel overhead after setup

## Implementation

```c
// Pipe Example
int pipefd[2];
pipe(pipefd);

if (fork() == 0) {
    close(pipefd[0]);  // Close read end
    write(pipefd[1], "Hello", 5);
    close(pipefd[1]);
} else {
    close(pipefd[1]);  // Close write end
    char buf[10];
    read(pipefd[0], buf, 10);
    close(pipefd[0]);
}

// Message Queue
mqd_t mq = mq_open("/queue", O_CREAT | O_RDWR, 0644, &attr);
mq_send(mq, "msg", 3, 0);
char buffer[100];
mq_receive(mq, buffer, 100, NULL);

// Shared Memory
int shmid = shmget(IPC_PRIVATE, 4096, IPC_CREAT | 0666);
char *ptr = (char*)shmat(shmid, NULL, 0);
strcpy(ptr, "data");
shmdt(ptr);
```

## Examples

**Python Pipes**:
```python
from multiprocessing import Pipe

parent_conn, child_conn = Pipe()
if fork() == 0:
    child_conn.send("Hello")
else:
    print(parent_conn.recv())
```

**Java Shared Memory**:
```java
import java.nio.MappedByteBuffer;
import java.nio.channels.FileChannel;
import java.io.RandomAccessFile;

RandomAccessFile file = new RandomAccessFile("mem", "rw");
MappedByteBuffer buffer = file.getChannel()
    .map(FileChannel.MapMode.READ_WRITE, 0, 1024);
buffer.putInt(0, 42);
```

**JavaScript (Node.js) Message Queue**:
```javascript
const { fork } = require('child_process');

const child = fork('child.js');
child.send({ hello: 'world' });
child.on('message', (msg) => console.log(msg));
```

## Key Takeaways

1. **Pipes**: Simple, unidirectional, limited size
2. **Message Queues**: Persistent, typed messages, kernel-managed
3. **Shared Memory**: Fastest, requires manual synchronization
4. **Performance**: Shared Memory > Pipes > Message Queues
5. **Ease of use**: Message Queues > Pipes > Shared Memory

## Interview Focus

**Common Questions**:
1. Compare pipes, message queues, and shared memory
2. When would you use each mechanism?
3. Why is shared memory fastest?
4. What are limitations of pipes?
5. How do you synchronize shared memory access?

**Coding Questions**:
- Implement producer-consumer with pipes
- Create shared memory segment
- Use message queue for communication
