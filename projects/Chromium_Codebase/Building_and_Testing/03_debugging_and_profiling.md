# Debugging & Profiling

**[⬅️ Back to Building Overview](00_building_and_testing.md)**

---

## 🐛 Debugging with GDB/LLDB

Since Chromium is C++, standard debuggers work, but the size of the project requires some tricks.

### 1. Setup
*   **Linux**: `gdb`
*   **Mac**: `lldb`
*   **Build Args (`args.gn`)**:
    ```python
    is_debug = true
    symbol_level = 1 # or 2 for full symbols (very slow link time)
    blink_symbol_level = 1
    ```

### 2. Attaching to Processes
Chromium spawns many processes. You usually want to debug the **Renderer** or **Browser**.

*   **Find the PID**:
    ```bash
    ps aux | grep chrome
    # Look for --type=renderer or the main process
    ```
*   **Attach**:
    ```bash
    gdb -p <PID>
    ```

### 3. Debugging Startup (The "Waiting" Trick)
If you need to debug code that runs *before* you can attach:
1.  Add this code where you want to pause:
    ```cpp
    #include "base/debug/debugger.h"
    base::debug::WaitForDebugger(60, true);
    ```
2.  Run Chrome. It will hang.
3.  Attach your debugger.

### 4. Pretty Printers
Chromium has GDB scripts to make STL and Blink types readable.
*   **Load**: `source /path/to/chromium/src/tools/gdb/gdbinit`

---

## 🏎️ Profiling with Perfetto

**Perfetto** is the system-wide profiling tool used by Chromium and Android. It replaces the old `chrome://tracing`.

### 1. Recording a Trace
1.  Open `chrome://tracing` (Legacy) or the [Perfetto UI](https://ui.perfetto.dev/).
2.  **Categories**: Select what you want to record:
    *   `blink`: Rendering logic.
    *   `v8`: JS execution.
    *   `toplevel`: Main loop tasks.
    *   `mojom`: IPC messages.
3.  Click **Record**, do your action, click **Stop**.

### 2. Analyzing
*   **Slices**: Look for long bars. Each bar represents a function call or a task.
*   **Flows**: Arrows connecting slices show causal relationships (e.g., IPC sent -> IPC received).

### 3. Adding Your Own Trace Events
To see your code in the trace:

```cpp
#include "base/trace_event/trace_event.h"

void MyFunction() {
  TRACE_EVENT0("category_name", "MyFunction");
  // ... code ...
}
```

---

## 📜 Logging

Sometimes `printf` is king.

*   **Info**: `LOG(INFO) << "Loaded " << count << " items.";`
*   **Warning**: `LOG(WARNING) << "Something weird happened.";`
*   **Error**: `LOG(ERROR) << "Something bad happened.";`
*   **Debug Only**: `DLOG(INFO) << "Only prints in debug builds.";`
*   **Verbose**: `VLOG(1) << "Detailed logs.";` (Enable with `--v=1`).

**Where do logs go?**
*   **Linux/Mac**: stderr (your terminal).
*   **Windows**: Debug Output (use DebugView or run with `--enable-logging --v=1`).
