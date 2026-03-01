# Architecture

**[⬅️ Back to Overview](../00_overview.md)**

---

## 🏗️ Architectural Concepts

This section covers the high-level design and core architectural components of Chromium.

### 1. [Multi-Process Architecture](01_multi_process_model.md)
*   **Browser Process**: Coordinates the application, handles I/O, and manages other processes.
*   **Renderer Process**: Renders web pages (Blink, V8). Sandboxed for security.
*   **GPU Process**: Handles GPU tasks.
*   **Utility Processes**: Network service, Audio service, etc.

### 2. [Threading & Task Scheduling](02_threading_and_tasks.md)
*   **Message Loops**: `base::MessageLoop`.
*   **Task Posting**: `base::PostTask`.
*   **Mojo**: Inter-process communication (IPC).

### 5. [V8 JavaScript Engine](05_v8_engine.md)
*   **Ignition & TurboFan**: Interpretation and compilation pipeline.
