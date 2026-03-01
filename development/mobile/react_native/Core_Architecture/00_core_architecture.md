# Core Architecture

This chapter explores the internal execution model of React Native. Unlike standard web development, React Native operates across multiple threads and relies on specific communication bridges or interfaces (JSI) to talk to the host operating system.

## In this chapter, you will learn:

*   **[1.1 Threads and Processes](01_threads_and_processes.md)**
    *   Understanding the JS Thread, UI (Main) Thread, and Shadow Thread.
    *   How thread blocking impacts performance.

*   **[1.2 The Bridge and JSI](02_the_bridge_and_jsi.md)**
    *   The legacy asynchronous JSON bridge vs. the modern synchronous JSI.
    *   How the "New Architecture" (Fabric) changes the game.

*   **[1.3 Rendering Pipeline](03_rendering_pipeline.md)**
    *   From React Element to Shadow Tree to Native View.
    *   The reconciliation process in a native context.

*   **[1.4 Hermes Engine](04_hermes_engine.md)**
    *   Why a specialized JavaScript engine is needed.
    *   Bytecode precompilation and startup time optimization.
