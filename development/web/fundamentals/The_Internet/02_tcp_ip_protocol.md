# TCP/IP & The Three-Way Handshake 🤝

**TCP (Transmission Control Protocol)** is the foundation of reliable communication on the internet. Unlike UDP (User Datagram Protocol), which acts like a "fire and forget" mechanism, TCP guarantees that packets are delivered in order and without errors.

---

## 1. The TCP Segment Structure 📦

A TCP segment consists of a header and a data section. Key fields include:

*   **Source/Destination Port**: Addressing the specific application.
*   **Sequence Number (SEQ)**: Tracks the order of bytes sent.
*   **Acknowledgment Number (ACK)**: The next byte expected from the sender.
*   **Flags**:
    *   `SYN` (Synchronize): Initiates a connection.
    *   `ACK` (Acknowledgment): Confirms receipt.
    *   `FIN` (Finish): Terminates a connection.
    *   `RST` (Reset): Aborts a connection.
*   **Window Size**: For flow control (how much data the receiver can accept).

---

## 2. The Three-Way Handshake (Connection Establishment) 🔗

Before data is exchanged, a virtual connection must be established. This is a 3-step process.

### Step 1: SYN (Client -> Server)
The client wants to connect. It sends a segment with:
*   Flag: `SYN=1`
*   SEQ: `x` (Random initial sequence number)
*   State: Client enters `SYN_SENT`.

### Step 2: SYN-ACK (Server -> Client)
The server receives the SYN. It acknowledges the client and sends its own SYN.
*   Flags: `SYN=1`, `ACK=1`
*   ACK: `x + 1` (Expecting the next byte from client)
*   SEQ: `y` (Server's random initial sequence number)
*   State: Server enters `SYN_RCVD`.

### Step 3: ACK (Client -> Server)
The client acknowledges the server's SYN.
*   Flag: `ACK=1`
*   ACK: `y + 1`
*   SEQ: `x + 1`
*   State: Client enters `ESTABLISHED`. Server enters `ESTABLISHED` upon receipt.

> **Analogy**:
> 1.  **Alice**: "Bob, are you there? (SYN)"
> 2.  **Bob**: "I hear you Alice, are you ready? (SYN-ACK)"
> 3.  **Alice**: "Yes, I'm ready. (ACK)"

---

## 3. Reliability Features 🛡️

### Flow Control (Sliding Window)
Prevents a fast sender from overwhelming a slow receiver. The receiver advertises a `Window Size` in every ACK packet, telling the sender how many bytes it can buffer. If the window creates 0, the sender stops.

### Congestion Control
Prevents the network backbone from collapsing.
*   **Slow Start**: Start sending small amounts of data, exponentially increase if successful.
*   **Congestion Avoidance**: If a packet is lost (timeout), TCP assumes network congestion and drastically cuts the transmission rate, then grows linearly.

### Connection Termination (Four-Way Handshake)
1.  **Client**: Sends `FIN`.
2.  **Server**: Sends `ACK`. (Server can still send data if needed).
3.  **Server**: Sends `FIN` (When done).
4.  **Client**: Sends `ACK`.

---

## 4. TCP vs UDP 🥊

| Feature | TCP | UDP |
| :--- | :--- | :--- |
| **Connection** | Connection-oriented (Handshake) | Connectionless |
| **Reliability** | Guaranteed delivery, ordered | Best effort, packet loss possible |
| **Speed** | Slower (overhead) | Fast (low overhead) |
| **Use Cases** | Web (HTTP), Email (SMTP), File Transfer (FTP) | Streaming, VoIP, Gaming, DNS |
