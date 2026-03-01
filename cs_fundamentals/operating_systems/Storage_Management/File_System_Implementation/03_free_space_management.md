# Free Space Management\n\n## Free Space Management

### Overview

Bit vector, linked list, grouping, counting

### Implementation

```c
typedef struct {
    int mode;
    int links;
    int uid;
    int gid;
    off_t size;
    time_t atime;
    time_t mtime;
    time_t ctime;
    blk_t blocks[15];  // Direct and indirect blocks
} Inode;

Inode *get_inode(int inode_num) {
    // Read inode from disk
    return inode;
}
```

### Mechanism

How this file system feature works.

**Python Simulation**:
```python
class FileSystem:
    def __init__(self):
        self.inodes = {}
        self.data_blocks = {}
        
    def allocate_block(self):
        # Find free block
        return block_num
        
    def read_file(self, inode):
        blocks = inode['blocks']
        data = b''.join(self.data_blocks[b] for b in blocks)
        return data
```

### Performance

Trade-offs and optimization strategies.

## Key Takeaways

1. File system structure balances performance and reliability
2. Allocation methods affect fragmentation and access speed
3. Free space management impacts allocation efficiency
4. Journaling ensures consistency after crashes

## Interview Focus

1. Explain file system implementation details
2. Compare allocation methods
3. How does journaling work?
4. Calculate disk blocks needed for file
