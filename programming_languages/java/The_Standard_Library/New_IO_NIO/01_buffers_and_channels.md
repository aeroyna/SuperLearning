# Buffers and Channels: Deep Dive

## 1. Heap vs. Direct Buffers

`ByteBuffer` comes in two flavors.

### 1.1 HeapByteBuffer
*   **Allocation:** `ByteBuffer.allocate(1024)`.
*   **Location:** Java Heap (Subject to GC).
*   **IO Mechanism:** When writing to a channel, the JVM must **copy** this buffer to a temporary native memory address because the OS cannot read directly from the Java Heap (GC might move it!).
*   **Cost:** Double copy overhead.

### 1.2 DirectByteBuffer
*   **Allocation:** `ByteBuffer.allocateDirect(1024)`.
*   **Location:** Native Memory (Off-Heap).
*   **IO Mechanism:** The OS reads directly from this address.
*   **Cost:** Expensive to allocate/deallocate. Not GC'd automatically (requires cleaner).
*   **Use Case:** Long-lived buffers for high-performance network/file IO.

## 2. Zero-Copy (FileChannel.transferTo)
Normally, sending a file to a network socket involves:
1.  Disk -> Kernel Buffer (DMA).
2.  Kernel Buffer -> User Buffer (CPU copy).
3.  User Buffer -> Kernel Socket Buffer (CPU copy).
4.  Socket Buffer -> NIC (DMA).

**Zero-Copy (`transferTo`):**
1.  Disk -> Kernel Buffer.
2.  Kernel Buffer -> NIC.
*   The data never touches the JVM or User Space. It saves CPU cycles and context switches.

```java
fileChannel.transferTo(0, fileChannel.size(), socketChannel);
```

## 3. Memory Mapped Files (`MappedByteBuffer`)
Allows mapping a file region directly into virtual memory.
*   The OS handles page swapping.
*   Reading the buffer is as fast as reading RAM.
*   Crucial for database engines (like Cassandra/Kafka) and high-speed logging.
