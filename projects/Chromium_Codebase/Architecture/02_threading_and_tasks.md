# Threading and Task Scheduling

**[⬅️ Back to Architecture Overview](00_architecture.md)**

---

## 🧵 The Golden Rule

> **Start Fast, Stay Responsive.**

To keep the UI smooth (60fps), the main thread must **never** block. Chromium enforces this by restricting blocking I/O (file, network) on the UI thread. If you try, the code will crash with a `base::ScopedAllowBlocking` assertion failure in debug builds.

---

## 🏗️ The Threading Infrastructure

Chromium moved from a dedicated `TaskScheduler` to a unified `base::ThreadPool`.

### 1. `base::ThreadPool`
The engine that runs tasks that don't need to be on a specific thread (like the UI thread).
*   **Usage**: "I have some work, run it whenever."
*   **Traits**: You annotate tasks with `base::TaskTraits` to hint how they should be scheduled.
    *   `base::TaskPriority::USER_BLOCKING`: Highest priority. Blocks user interaction (e.g., loading app data).
    *   `base::TaskPriority::USER_VISIBLE`: Result is visible (e.g., rendering a thumbnail).
    *   `base::TaskPriority::BEST_EFFORT`: Background tasks (e.g., indexing, reporting metrics).
    *   `base::MayBlock()`: Crucial. Tells the pool this task might block (File I/O). The pool may spawn more threads to compensate.

**Example:**
```cpp
base::ThreadPool::PostTask(
    FROM_HERE,
    {base::MayBlock(), base::TaskPriority::USER_VISIBLE},
    base::BindOnce(&SaveFileToDisk, data));
```

### 2. `content::BrowserThread`
Known, named threads in the Browser Process.
*   **UI Thread (`content::BrowserThread::UI`)**: The main thread. Updates windows, processes input.
*   **IO Thread (`content::BrowserThread::IO`)**: Handles IPC messages and network triggers.

**Posting to the UI Thread:**
```cpp
content::GetUIThreadTaskRunner({})->PostTask(
    FROM_HERE, base::BindOnce(&UpdateUI));
```
*Or purely within the browser process:*
```cpp
// Deprecated style but still common
content::BrowserThread::PostTask(
    content::BrowserThread::UI, FROM_HERE, base::BindOnce(&UpdateUI));
```

---

## 🔄 Sequences vs. Threads

Chromium prefers **Sequences** over raw threads for thread safety.

*   **Thread**: A physical OS thread.
*   **Sequence**: A virtual thread. Tasks posted to a sequence run **serially** (one after another), but can hop between different physical threads.
*   **Why?**: It provides thread safety (no race conditions between tasks on the same sequence) without the overhead of dedicating a physical thread to every component.

**Creating a Sequence:**
```cpp
scoped_refptr<base::SequencedTaskRunner> task_runner =
    base::ThreadPool::CreateSequencedTaskRunner(
        {base::MayBlock(), base::TaskPriority::USER_VISIBLE});

task_runner->PostTask(FROM_HERE, base::BindOnce(&Step1));
task_runner->PostTask(FROM_HERE, base::BindOnce(&Step2)); // Guaranteed to run after Step1
```

---

## ⚠️ Common Pitfalls

1.  **Blocking the UI Thread**: Reading a file or waiting for a mutex on the UI thread.
2.  **Use-After-Free**: Passing a raw pointer `this` to a task that runs after the object is destroyed.
    *   **Fix**: Use `base::WeakPtr`.
    ```cpp
    base::ThreadPool::PostTask(
        FROM_HERE,
        base::BindOnce(&MyClass::DoWork, weak_ptr_factory_.GetWeakPtr()));
    ```
3.  **Thread Hopping**: Accessing a member variable on the UI thread *and* a background thread without protection.
    *   **Fix**: use `SEQUENCE_CHECKER(sequence_checker_)` to assert code runs on the correct sequence.
