# HTTPS: TLS/SSL Handshake Explained 🔐

**HTTPS (Hypertext Transfer Protocol Secure)** is HTTP protocol encrypted over **TLS (Transport Layer Security)**. It ensures confidentiality, integrity, and authentication.

> **Note**: SSL (Secure Sockets Layer) is the deprecated predecessor. We strictly use TLS 1.2 or 1.3 now, but the term "SSL" persists colloquially.

---

## 1. Encryption Concepts 🔑

### Symmetric Encryption
*   **Mechanism**: Same key used for encryption and decryption.
*   **Pros**: Fast.
*   **Cons**: Key distribution (how do I give you the key without a hacker seeing it?).
*   **Usage**: Used for the actual data transfer *after* the handshake.

### Asymmetric Encryption (Public Key Cryptography)
*   **Mechanism**: A pair of keys. **Public Key** (encrypts) and **Private Key** (decrypts).
*   **Pros**: Secure key exchange.
*   **Cons**: Slow / computationally expensive.
*   **Usage**: Used only during the handshake to exchange the symmetric key.

---

## 2. The TLS 1.2 Handshake (Simplifed) 🤝

Before HTTP data is sent, the TLS handshake occurs (after TCP 3-way handshake).

1.  **Client Hello**:
    *   "I support TLS 1.2."
    *   "Here are the cipher suites I support (algorithms)."
    *   "Here is a random number (Client Random)."

2.  **Server Hello**:
    *   "Let's use TLS 1.2."
    *   "I choose this cipher suite."
    *   "Here is my **SSL Certificate** (containing Public Key)."
    *   "Here is a random number (Server Random)."

3.  **Authentication (Client Side)**:
    *   The browser validates the Certificate against its list of Trusted Root CAs (Certificate Authorities).
    *   Checks expiry, domain name match, and revocation status.

4.  **Key Exchange (Premaster Secret)**:
    *   The client generates a **Premaster Secret**.
    *   The client encrypts it using the server's **Public Key**.
    *   Sends the encrypted secret to the server.

5.  **Session Key Generation**:
    *   Both Client and Server now have: Client Random + Server Random + Premaster Secret.
    *   They use an algorithm to generate the **Session Key** (Symmetric Key).

6.  **Finished**:
    *   Both parties send a "Finished" message encrypted with the Session Key to verify.
    *   **Tunnel Established**: All subsequent HTTP traffic is encrypted with the Session Key.

---

## 3. TLS 1.3 (Faster & Safer) 🚀

TLS 1.3 (released 2018) optimized this process.
*   **1-RTT (Round Trip Time)**: The key exchange happens *during* the Hello phase, removing a full round trip.
*   **Zero-RTT (Resumption)**: If the client has visited before, it can send encrypted data in the *first packet*.
*   **Security**: Removed weak cipher suites (MD5, SHA-1).

---

## 4. The Chain of Trust ⛓️

How do you trust `google.com`?

1.  **Root CA**: The OS/Browser comes pre-installed with "Root Certificates" (e.g., DigiCert, Let's Encrypt). These are implicitly trusted.
2.  **Intermediate CA**: The Root CA signs an Intermediate CA.
3.  **Leaf Certificate**: The Intermediate CA signs the website's certificate.

If the chain is broken (e.g., self-signed certificate), the browser shows a "Not Secure" warning.
