# The Bridge and JSI (JavaScript Interface)

The mechanism by which JavaScript communicates with the Native platform is the defining characteristic of React Native's architecture.

## The Legacy Architecture: The Bridge

In the original React Native architecture, the JS Realm and the Native Realm were completely isolated. They communicated via an asynchronous "Bridge".

### How it works
1.  **Serialization:** A command in JS (e.g., `View.create(...)`) is serialized into JSON.
2.  **Transmission:** The JSON string is batched and sent across the bridge.
3.  **Deserialization:** The Native side decodes the JSON and executes the native method (e.g., `UIView` init).

### Bottlenecks
*   **Asynchronous:** You cannot return a value synchronously from Native to JS.
*   **Overhead:** JSON serialization/deserialization is expensive (CPU intensive).
*   **Congestion:** High-frequency events (like scroll events or animations) can flood the bridge, causing lag.

## The New Architecture: JSI (JavaScript Interface)

JSI is a completely new layer that replaces the Bridge. It allows JavaScript to hold references to C++ Host Objects and invoke methods on them *synchronously* and *directly*.

### Key Improvements
1.  **Synchronous Execution:** JS can call a C++ method and get the result immediately.
2.  **No Serialization:** Shared memory is used. No JSON overhead.
3.  **Interoperability:** Allows RN to use other JS engines easily (like V8 or Hermes) because JSI is an abstraction over the VM.

### TurboModules
Built on top of JSI, TurboModules allow Native Modules to be:
*   **Lazy Loaded:** Modules are loaded only when needed, speeding up app startup.
*   **Type-Safe:** CodeGen ensures JS and Native types match.

## Summary
*   **Old:** "Message passing" via JSON (Slow, Async).
*   **New:** Direct function invocation via C++ (Fast, Sync).
