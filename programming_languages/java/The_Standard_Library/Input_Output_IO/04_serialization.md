# Serialization: Deep Dive & Security

## 1. The Serialization Protocol
Java Serialization writes a graph of objects.
1.  Writes "Magic Number" and Version header.
2.  Writes Class Metadata (`serialVersionUID`, field names).
3.  Writes Data values recursively.

## 2. `serialVersionUID` Nuance
This ID is a hash of the class structure (methods, fields).
*   **Default:** Computed at runtime. Extremely brittle. Even compiling with a different `javac` version can change it.
*   **Explicit:** `private static final long serialVersionUID = 1L;`. Allows you to add methods without breaking compatibility with old serialized files.

## 3. Customizing Serialization
*   **`writeObject(ObjectOutputStream out)`**: You can implement this private method to control exactly how fields are written (e.g., encryption).
*   **`readObject(ObjectInputStream in)`**: Implement to verify invariants after deserialization (e.g., ensure `age` is positive).
*   **`readResolve()`**: For Singletons. Ensures deserialization returns the *existing* singleton instance rather than creating a new one.

## 4. The Security Nightmare
**Deserialization of untrusted data is a Remote Code Execution (RCE) vulnerability.**
*   **Gadget Chains:** An attacker sends a stream containing a malicious object graph. While the JVM deserializes it, it executes code (like `readObject` triggers). If common libraries (like Apache Commons Collections) are on the classpath, attackers can chain these triggers to execute shell commands.
*   **Defense:** Use **Serialization Filters** (Java 9+) to whitelist allowed classes, or avoid native serialization entirely (use JSON).
