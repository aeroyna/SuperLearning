# Thread Lifecycle

Understanding the lifecycle states of a thread is critical for debugging (interpreting thread dumps) and understanding why your application might be hanging or running slowly.

## 1. The 6 States of a Java Thread
Java defines 6 states in the `Thread.State` enum.

### 1. NEW
*   **Definition:** The thread object has been created (`new Thread()`), but the `start()` method has not yet been called.
*   **Internal:** The OS has not yet allocated a native thread context for this object. It's just a regular Java object on the heap.

### 2. RUNNABLE (The Active State)
*   **Definition:** The thread is executing in the JVM.
*   **Nuance:** It maps to the OS state **Ready** or **Running**.
    *   **Running:** The CPU is physically executing instructions for this thread right now.
    *   **Ready:** The thread is ready to run and waiting in the queue for the OS Scheduler to give it a time slice.
*   *Why this matters:* A thread in `RUNNABLE` state might actually be sitting idle if the system is overloaded (high Load Average).

### 3. BLOCKED (The Monitor Lock State)
*   **Definition:** The thread is waiting to acquire a **Monitor Lock** to enter a `synchronized` block or method.
*   **Scenario:** Thread A is inside a `synchronized` block. Thread B tries to enter. Thread B goes into `BLOCKED`.
*   **Exit:** Once Thread A releases the lock *and* the OS scheduler picks Thread B to acquire it.

### 4. WAITING (Infinite Wait)
*   **Definition:** The thread is waiting indefinitely for another thread to perform a particular action.
*   **Triggers:**
    *   `Object.wait()` (without timeout)
    *   `Thread.join()` (without timeout)
    *   `LockSupport.park()`
*   **Exit:** Only whenever `notify()`, `notifyAll()`, or `unpark()` is called.
*   *Danger:* If the notification is missed or never sent, this thread hangs forever (Deadlock/Livelock).

### 5. TIMED_WAITING (Finite Wait)
*   **Definition:** Waiting for another thread, but with a specified waiting time.
*   **Triggers:**
    *   `Thread.sleep(ms)`
    *   `Object.wait(ms)`
    *   `Thread.join(ms)`
*   **Exit:** Time expires OR notification is received.
*   *Use Case:* Polling, timeouts, rate limiting.

### 6. TERMINATED
*   **Definition:** The thread has completed execution.
*   **Cause:** The `run()` method finished normally, or an unhandled exception escaped `run()`.
*   **Nuance:** A terminated thread object still exists on the Heap until GC collects it, but it cannot be restarted (`start()` throws exception).

---

## 2. State Transition Diagram

```text
       [ NEW ]
          | start()
          v
     [ RUNNABLE ] <--------+
          |                |
          | (Scheduler)    |
          v                |
   (Executing Code) -------+
    /     |      \
   /      |       \  synchronized
  /       |        \ (Lock held by other)
 v        v         v
[TIMED] [WAITING] [BLOCKED]
[WAIT ]           
```

## 3. Debugging with Thread Dumps
When you take a Thread Dump (using `jstack` or VisualVM), you see these states.

*   **Healthy App:** Most threads in `WAITING` (idle in thread pool) or `RUNNABLE`.
*   **Hung App:** Many threads in `BLOCKED` (contention on a lock).
*   **CPU Spike:** Many threads in `RUNNABLE` actually doing work (check the stack trace to see what they are processing).

```