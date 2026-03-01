# File Operations\n\n## File Operations

### Overview

Create, read, write, delete, reposition

### Concept

File systems organize and manage data on storage devices.

### Structure

```c
typedef struct {
    char name[256];
    int type;
    int size;
    int permissions;
    time_t created;
    time_t modified;
    int inode_number;
} FileDescriptor;

int open(const char *path, int flags) {
    // Open file and return file descriptor
    return fd;
}
```

### Operations

**C Examples**:
```c
#include <fcntl.h>
#include <unistd.h>

int fd = open("file.txt", O_RDWR | O_CREAT, 0644);
write(fd, buffer, size);
read(fd, buffer, size);
close(fd);
```

**Java**:
```java
import java.io.*;

File file = new File("file.txt");
FileInputStream fis = new FileInputStream(file);
byte[] data = new byte[1024];
fis.read(data);
fis.close();
```

**Python**:
```python
with open('file.txt', 'r') as f:
    data = f.read()
# File automatically closed
```

### Properties

Key characteristics and design considerations.

## Key Takeaways

1. Files abstract storage devices
2. File operations are fundamental system calls
3. Directory structures organize files hierarchically
4. File systems balance performance and features

## Interview Focus

1. Explain file system organization
2. What are file operations?
3. Compare directory structures
4. How does file mounting work?
