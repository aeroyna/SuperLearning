# Multi-Process Architecture & IPC

**[⬅️ Back to Architecture Overview](00_architecture.md)**

---

## 🏛️ The Multi-Process Model

Chromium's architecture is defined by its use of multiple processes to ensure stability, security, and performance. Unlike early browsers that ran everything in a single process, Chromium splits functionalities into isolated processes.

### 1. Key Processes

*   **Browser Process (The "Kernel")**:
    *   **Count**: Exactly 1.
    *   **Role**: The main process that coordinates everything. It manages the application state, coordinates other processes, and handles network I/O (though network service is moving out) and file I/O.
    *   **Responsibilities**:
        *   Managing the UI (Omnibox, back/forward buttons).
        *   Spawning and managing child processes.
        *   Handling user input and routing it to the correct renderer.
        *   Implementing high-level features like Downloads, History, and Bookmarks.

*   **Renderer Process**:
    *   **Count**: Multiple (one per tab/site instance, dependent on process isolation policy).
    *   **Role**: Interprets HTML, CSS, and JavaScript to display a web page.
    *   **Components**:
        *   **Blink**: The rendering engine (Layout, Paint).
        *   **V8**: The JavaScript engine.
    *   **Security**: Runs inside a restrictive **Sandbox**. It has no direct access to the filesystem or OS windowing system. It must ask the Browser Process to perform privileged actions.

*   **GPU Process**:
    *   **Count**: Exactly 1.
    *   **Role**: Handles GPU tasks. It isolates the potentially unstable GPU drivers from the browser process.
    *   **Responsibility**: compositing and drawing the final pixels to the screen.

*   **Utility Processes**:
    *   **Count**: Variable.
    *   **Role**: Used for specific tasks to isolate them from the browser process.
    *   **Examples**: Network Service, Audio Service, Storage Service, Unzipping files.

---

## 🔌 Mojo IPC (Inter-Process Communication)

As Chromium split into more granular services ("Servicefication"), the need for a high-performance IPC system became critical. **Mojo** is that system.

### 1. What is Mojo?
Mojo is a collection of runtime libraries and a code generator that provides a platform-agnostic abstraction of common IPC primitives. It is:
*   **Faster**: ~3x faster than the legacy Chrome IPC.
*   **Simpler**: Uses an IDL (Interface Definition Language) to generate type-safe bindings for C++, Java, and JavaScript.

### 2. How it Works
*   **Message Pipe**: A bi-directional channel between two endpoints (ports).
*   **Bindings**: Generated code that allows you to call methods on an interface as if it were a local object.
*   **Remote**: A proxy object used to send messages to the other end of the pipe.
*   **Receiver**: The object that implements the interface and handles incoming messages.

### 3. Example: Mojo IDL (`.mojom`)
```mojom
// //content/common/renderer.mojom
module content.mojom;

interface Renderer {
  // Tells the renderer to create a new view.
  CreateView(CreateViewParams params);
};
```

### 4. Code Flow with Mojo
1.  **Browser Process**: Holds a `mojo::Remote<content::mojom::Renderer>`.
2.  **Call**: Browser calls `renderer_remote_->CreateView(params)`.
3.  **Transport**: Mojo serializes `params`, writes to the message pipe.
4.  **Renderer Process**: The `mojo::Receiver<content::mojom::Renderer>` reads the message, deserializes it, and calls `CreateView` on the implementation class.

---

## 🧵 Threading Model

Chromium is also highly multi-threaded within each process to keep the main thread responsive.

### 1. The Main Thread (UI Thread)
*   Found in the **Browser Process**.
*   Handles window management and user interface events.
*   **Rule**: Never perform blocking I/O on this thread.

### 2. The IO Thread
*   Found in the **Browser Process** (and others).
*   Handles IPC messages and network requests (historically, before Network Service).
*   **Confusion**: It's named "IO Thread" but mostly handles *IPC I/O*, not necessarily file I/O.

### 3. Dedicated Worker Threads
*   `base::ThreadPool`: A pool of threads for running blocking tasks (File I/O, heavy computation).
*   **Posting Tasks**:
    ```cpp
    base::ThreadPool::PostTask(
        FROM_HERE,
        {base::MayBlock(), base::TaskPriority::USER_VISIBLE},
        base::BindOnce(&MyFunction));
    ```

### 4. `BrowserThread` Identifier
In the browser process, threads are identified by an enum:
*   `content::BrowserThread::UI`
*   `content::BrowserThread::IO`

**Check your thread:**
```cpp
DCHECK_CURRENTLY_ON(content::BrowserThread::UI);
```
