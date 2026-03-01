# Performance and Optimization

Performance is the primary reason developers choose React Native over Hybrid/WebView apps, but it is easy to degrade if you don't understand the constraints.

## In this chapter, you will learn:

*   **[6.1 List Virtualization](01_list_virtualization.md)**
    *   Why standard `.map()` fails for long lists.
    *   How `FlatList` recycles views.
    *   The "Blank Space" problem.

*   **[6.2 Memory Leaks and Profiling](02_memory_leaks_and_profiling.md)**
    *   Identifying leaks with Android Studio Profiler and Xcode Instruments.
    *   Common closure and timer leaks.

*   **[6.3 Bundle Optimization](03_bundle_optimization.md)**
    *   Reducing the JavaScript bundle size.
    *   Inline Requires and Tree Shaking.
