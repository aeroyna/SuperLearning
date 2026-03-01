# DMA (Direct Memory Access)\n\n## DMA (Direct Memory Access)

### Overview

Bypass CPU for data transfer

### Mechanism

```c
typedef struct {
    void *buffer;
    size_t size;
    int device_id;
    int status;
} IORequest;

void submit_io(IORequest *req) {
    // Queue I/O request
    // Device driver handles execution
}

void io_completion_handler(int device) {
    // Handle I/O completion interrupt
}
```

### Implementation

**C Example**:
```c
// DMA transfer
void dma_transfer(void *src, void *dest, size_t size) {
    dma_controller->src_addr = src;
    dma_controller->dest_addr = dest;
    dma_controller->count = size;
    dma_controller->control = DMA_START;
    
    // Wait for completion
    wait_for_interrupt();
}
```

**Java**:
```java
class IOManager {
    void asyncRead(ByteBuffer buffer) {
        channel.read(buffer, attachment, new CompletionHandler() {
            public void completed(Integer result, Object attachment) {
                // I/O complete
            }
        });
    }
}
```

### Performance

Impact on system throughput and latency.

## Key Takeaways

1. I/O hardware provides interface between devices and CPU
2. Buffering and caching improve performance
3. DMA reduces CPU involvement in I/O
4. Kernel I/O subsystem manages device access

## Interview Focus

1. Explain I/O hardware components
2. How does DMA work?
3. What is the purpose of buffering?
4. Compare blocking vs non-blocking I/O
