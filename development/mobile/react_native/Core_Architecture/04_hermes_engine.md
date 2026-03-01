# Hermes Engine

Hermes is an open-source JavaScript engine optimized specifically for running React Native apps on Android (and now iOS).

## Why not V8 or JavaScriptCore (JSC)?
Standard engines like V8 (Chrome) and JSC (Safari) are optimized for JIT (Just-In-Time) compilation and peak performance in long-running web sessions. Mobile apps have different constraints:
*   **Startup Time:** Users expect immediate load.
*   **Memory Usage:** Mobile devices have limited RAM.
*   **Download Size:** APK/IPA size matters.

## Key Features of Hermes

### 1. Ahead-of-Time (AOT) Compilation
Instead of parsing and compiling JavaScript on the device every time the app starts, Hermes moves this to the **build time**.
*   **Process:** During the build, the Metro bundler produces JavaScript, which Hermes then compiles into **Bytecode**.
*   **Benefit:** The device loads compact Bytecode directly. No parsing overhead. Massive startup speedup.

### 2. No JIT (Just-In-Time) Compiler
Hermes discards the JIT compiler.
*   **Why?** JIT warms up code to make it faster *eventually*, but it consumes significant memory and CPU during startup.
*   **Benefit:** Lower memory footprint and consistent performance from the start.

### 3. Garbage Collection (GenGC)
Hermes uses a custom Garbage Collector optimized for mobile patterns (allocating many short-lived objects).
*   **Virtual Address Space compaction:** It can move memory around to prevent fragmentation.
*   **On-Demand Paging:** It maps bytecode files into memory, loading only what is executed (reducing RAM usage).

## Enabling Hermes
In `android/app/build.gradle`:
```groovy
project.ext.react = [
    enableHermes: true
]
```
(It is now the default in new React Native versions).
