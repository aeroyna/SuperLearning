# JIT Compiler: Tiered Compilation

The Just-In-Time (JIT) compiler is the heart of Java's performance.

## 1. Interpreted vs Compiled
*   **Interpreter:** Starts fast, runs slow. Reads bytecode instruction by instruction.
*   **Compiler:** Takes time to run, but produces highly optimized native Assembly.

## 2. HotSpot Detection
The JVM counts method invocations.
*   **Warm:** If a method runs > 1500 times (configurable), it's queued for **C1 Compilation**.
*   **Hot:** If it runs > 10,000 times, it's queued for **C2 Compilation**.

## 3. Tiered Compilation (Default)
1.  **Tier 0:** Interpreter.
2.  **Tier 1-3 (C1 Client Compiler):** Quick optimization. Generates native code with profiling instrumentation. Fast to compile, decent speed.
3.  **Tier 4 (C2 Server Compiler):** Aggressive optimization based on profiling data (branch prediction, inlining). Slow to compile, maximum speed.

## 4. Deoptimization
Unlike C++, JIT allows **speculative optimization**.
*   *Assumption:* "This `if` statement has never been true." -> Remove the code.
*   *Reality Check:* Suddenly the `if` becomes true!
*   *Action:* **Deoptimize**. Throw away the native code, jump back to the Interpreter, and handle the case.