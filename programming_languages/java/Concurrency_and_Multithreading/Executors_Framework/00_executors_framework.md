# Executors Framework

Manually creating threads (`new Thread().start()`) is poor practice in large applications. The **Executors Framework** (`java.util.concurrent.ExecutorService`) solves this by decoupling **Task Submission** from **Task Execution**.

## In this chapter, you will learn:
*   [**Thread Pools**](01_thread_pools.md): Fixed vs Cached pools, internal queueing, and rejection policies.
*   [**Callable and Future**](02_callable_and_future.md): Returning values from threads.
*   [**CompletableFuture**](03_completablefuture.md): Non-blocking, reactive asynchronous pipelines.
