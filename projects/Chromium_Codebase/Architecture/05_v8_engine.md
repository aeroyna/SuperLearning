# V8 JavaScript Engine

**[⬅️ Back to Architecture Overview](00_architecture.md)**

---

## 🚀 V8 Architecture

V8 is Google's open-source high-performance JavaScript and WebAssembly engine. It implements ECMAScript and WebAssembly.

### 1. The Pipeline

The journey of JavaScript code in V8:
1.  **Source Code**: The raw JS string.
2.  **Parser**: Converts source into an Abstract Syntax Tree (AST).
3.  **Ignition (Interpreter)**:
    *   Walks the AST and generates **Bytecode**.
    *   Executes the bytecode using a register-based interpreter.
    *   **Goal**: Start running code as fast as possible with low memory overhead.
    *   **Feedback Vector**: While running, it collects type information (e.g., "This function is always called with Integers").
4.  **TurboFan (Optimizing Compiler)**:
    *   Takes the Bytecode + Feedback Vector.
    *   Generates highly optimized **Machine Code** for the specific CPU architecture.
    *   **Speculative Optimization**: It assumes the types will stay the same.
    *   **Deoptimization**: If assumptions fail (e.g., a String is passed instead of an Integer), it "bails out" back to Ignition/Bytecode.

### 2. Memory Management (Garbage Collection)
V8 uses a generational garbage collector (Orinoco).
*   **The Heap**: Divided into **New Space** (young generation) and **Old Space** (old generation).
*   **Minor GC (Scavenger)**: Fast. Moves live objects from New Space to Old Space.
*   **Major GC (Mark-Sweep-Compact)**: Slower. Cleans up the Old Space. It is incremental and concurrent to avoid pausing the main thread for too long.

---

## 🤝 V8 and Chromium

How do they talk?

### 1. Bindings
*   **IDL**: Web APIs (like `document.createElement`) are defined in WebIDL files.
*   **Code Generation**: The build system generates C++ "glue code" that wraps Blink C++ objects into V8 JavaScript objects.

### 2. Isolates & Contexts
*   **Isolate**: An instance of the V8 engine. Usually one per thread (one per Renderer Process main thread). It has its own heap and garbage collector.
*   **Context**: An environment for executing JavaScript code. Each `<iframe>` or window has its own Context. This ensures that global variables in one iframe don't leak into another.

### 3. Handles
*   **`v8::Local<T>`**: A pointer to a V8 object on the stack. Managed by `HandleScope`.
*   **`v8::Persistent<T>`**: A pointer to a V8 object that lives longer than a function call. Must be explicitly managed.
