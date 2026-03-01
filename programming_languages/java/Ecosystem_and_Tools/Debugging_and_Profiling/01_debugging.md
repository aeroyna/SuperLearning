# Debugging and Profiling

## JDB (Java Debugger)
The command-line debugger included with the JDK. Similar to GDB for C++.

### Basic Usage
1.  **Compile with debug info:** `javac -g MyApp.java`
2.  **Start JDB:** `jdb MyApp`
3.  **Set Breakpoint:** `stop at MyApp:10`
4.  **Run:** `run`
5.  **Inspect:** `print myVar`

## VisualVM
A visual tool for monitoring JVM applications. It allows you to see:
*   **Heap Dump:** Analyze memory usage and find leaks.
*   **Thread Dump:** Diagnose deadlocks and CPU spikes.
*   **CPU Profiler:** Find slow methods.

*Note: VisualVM is sometimes bundled with the JDK or can be downloaded separately.*
