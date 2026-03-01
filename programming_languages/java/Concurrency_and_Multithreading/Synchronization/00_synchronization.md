# Synchronization

In a multithreaded environment, threads often need to access shared resources (objects, files, variables). Without coordination, this leads to chaos. Synchronization is the mechanism to ensure that only one thread accesses a critical resource at a time.

## In this chapter, you will learn:
*   [**Race Conditions**](01_race_conditions_and_critical_sections.md): How state corruption happens at the CPU level.
*   [**Synchronized Keyword**](02_synchronized_keyword.md): Intrinsic locks, reentrancy, and the visibility guarantee.
*   [**Deadlocks**](03_deadlocks_and_liveness.md): The Coffman conditions and prevention strategies.
*   [**Volatile Keyword**](04_volatile_keyword.md): Memory visibility without atomicity.
*   [**Wait/Notify**](05_wait_notify_notifyall.md): The low-level inter-thread communication mechanism.
