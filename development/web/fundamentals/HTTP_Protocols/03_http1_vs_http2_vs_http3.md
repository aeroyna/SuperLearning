# Evolution: HTTP/1.1 vs HTTP/2 vs HTTP/3 🚀

The web has evolved to demand higher performance and lower latency. The protocol transporting this data has changed drastically.

---

## 1. HTTP/1.1 (1997) - The Standard 📜

### Mechanics
*   **Text-based**: Plain text headers (readable by humans).
*   **Keep-Alive**: Connections are persistent by default (reusing TCP connection).

### The Problem: Head-of-Line (HOL) Blocking 🛑
In HTTP/1.1, browsers open ~6 parallel TCP connections per domain. If one request takes a long time (e.g., a large image), other requests on that connection are blocked waiting for it to finish.
*   **Workaround**: Domain sharding (cdn1.com, cdn2.com) and image sprites.

---

## 2. HTTP/2 (2015) - The Multiplexer 🔀

Built on Google's SPDY, HTTP/2 addressed HOL blocking at the application layer.

### Key Features
1.  **Binary Protocol**: No longer text-based. More efficient to parse, less error-prone.
2.  **Multiplexing**: Allows multiple requests and responses to be sent **in parallel** over a single TCP connection.
    *   *Analogy*: HTTP/1.1 is like a single checkout lane. HTTP/2 is like a grocery store where items from different customers are interleaved on the conveyor belt but sorted correctly at the end.
3.  **Header Compression (HPACK)**: Headers (User-Agent, Cookies) are massive. HPACK compresses them and uses a lookup table for common headers so they aren't resent every time.
4.  **Server Push**: The server can send resources (style.css) before the client asks for them.

### Remaining Problem
**TCP HOL Blocking**: HTTP/2 solved blocking at the HTTP layer, but it still runs on TCP. If **one TCP packet** is lost, the Operating System pauses the entire stream to wait for retransmission, blocking *all* multiplexed streams inside it.

---

## 3. HTTP/3 (2022) - The QUIC One ⚡

HTTP/3 runs on **QUIC** (Quick UDP Internet Connections), a protocol built on top of **UDP**, not TCP.

### Key Features
1.  **Runs on UDP**: No TCP handshake overhead.
2.  **Solves TCP HOL Blocking**: Streams are independent. If a packet for Stream A is lost, Stream B continues processing. The concept of "order" is maintained per stream, not globally.
3.  **Zero-RTT Connection Setup**: TLS 1.3 is built-in. Handshake and encryption setup happen instantly.
4.  **Connection Migration**: If you switch from Wi-Fi to 4G, your IP changes. TCP connections break. QUIC uses a **Connection ID**, so the connection survives the network switch seamlessy.

### Comparison Table

| Feature | HTTP/1.1 | HTTP/2 | HTTP/3 |
| :--- | :--- | :--- | :--- |
| **Transport** | TCP | TCP | UDP (QUIC) |
| **Encoding** | Text (ASCII) | Binary | Binary |
| **Multiplexing**| No (Requests Queued) | Yes (One Conn) | Yes (Indep. Streams) |
| **Encryption** | Optional (TLS) | Application (TLS) | Built-in (TLS 1.3) |
| **Header Compression** | None | HPACK | QPACK |
