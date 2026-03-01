# Selectors and Async IO

Selectors are the key to Java NIO's high-performance networking capabilities. They allow a single thread to monitor multiple **Channels** of input.

## 1. The Problem with Blocking IO
In traditional (blocking) IO, if you want to handle 1000 client connections, you typically need 1000 threads. Each thread blocks (sleeps) waiting for data to arrive. Threads are expensive resources (memory, context switching).

## 2. The Selector Solution (Multiplexing)
A `Selector` allows a single thread to ask: *"Which of these 1000 channels have data ready to be read?"*

*   The thread calls `select()`.
*   It blocks until at least one channel is ready.
*   When it wakes up, it gets a list of "ready" channels.
*   It processes them sequentially.

## 3. How it Works

### 3.1 Components
1.  **SelectableChannel:** A channel that can be used with a selector (e.g., `SocketChannel`, `ServerSocketChannel`).
2.  **Selector:** The multiplexer.
3.  **SelectionKey:** Represents the registration of a Channel with a Selector.

### 3.2 Registration
You register a channel with a selector for a specific "interest set" (operations you want to monitor):
*   `OP_READ`: Data is ready to read.
*   `OP_WRITE`: Buffer is ready to write.
*   `OP_CONNECT`: Successfully connected to server.
*   `OP_ACCEPT`: Incoming connection pending.

```java
// Set channel to non-blocking mode (Mandatory for Selectors)
channel.configureBlocking(false);

// Register
SelectionKey key = channel.register(selector, SelectionKey.OP_READ);
```

### 3.3 The Loop
```java
while(true) {
    int readyChannels = selector.select(); // Blocks until something is ready
    if(readyChannels == 0) continue;

    Set<SelectionKey> selectedKeys = selector.selectedKeys();
    Iterator<SelectionKey> keyIterator = selectedKeys.iterator();

    while(keyIterator.hasNext()) {
        SelectionKey key = keyIterator.next();

        if(key.isAcceptable()) {
            // Accept connection
        } else if (key.isReadable()) {
            // Read data
        }
        
        keyIterator.remove(); // Crucial: Remove key to prevent reprocessing
    }
}
```

## 4. Asynchronous File Channel (Java 7)
Java 7 added `AsynchronousFileChannel` which allows file operations to execute entirely in the background, invoking a `CompletionHandler` callback when finished.

```java
AsynchronousFileChannel fileChannel = AsynchronousFileChannel.open(path, StandardOpenOption.READ);

ByteBuffer buffer = ByteBuffer.allocate(1024);
long position = 0;

fileChannel.read(buffer, position, buffer, new CompletionHandler<Integer, ByteBuffer>() {
    @Override
    public void completed(Integer result, ByteBuffer attachment) {
        System.out.println("Read finished: " + result + " bytes");
    }
    @Override
    public void failed(Throwable exc, ByteBuffer attachment) {
        System.err.println("Read failed");
    }
});
```

## 5. Summary
*   Use **Stream IO** for simple, sequential tasks.
*   Use **NIO (Selectors)** for high-concurrency network servers (chat, games, web servers).
*   Use **NIO.2 (Files/Path)** for all modern file system operations.