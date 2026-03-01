# Byte Streams: Deep Dive

## 1. The Cost of I/O
I/O is the slowest operation a computer performs.
*   **CPU:** Nanoseconds.
*   **RAM:** Nanoseconds.
*   **Disk/Network:** Milliseconds (Millions of times slower).

## 2. Why Buffering Matters
When you call `fos.write(byte)`, it creates a System Call (Context Switch to Kernel Mode).
*   **Unbuffered:** Write 1000 bytes -> 1000 System Calls.
*   **Buffered (8KB):** Write 1000 bytes -> 0 System Calls (stored in Java heap buffer). Write 8192 bytes -> 1 System Call.

**Flush:** `flush()` forces the buffer to empty into the OS. Closing a stream auto-flushes.

## 3. The "Mark/Reset" Contract
Some InputStreams support "rewinding".
*   `mark(readlimit)`: "Remember this position."
*   `reset()`: "Go back to the mark."
*   *Support:* `BufferedInputStream` supports it (by keeping read bytes in memory). `FileInputStream` does not. Always check `markSupported()`.

## 4. Closing Streams
*   **Memory Leaks:** Failing to close a stream leaks **File Descriptors** (OS handles). The OS has a limit (e.g., 1024 open files). Exceeding this crashes the app (`Too many open files`).
*   **Best Practice:** Always use `try-with-resources`.
