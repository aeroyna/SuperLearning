# CompletableFuture

`Future` has a major limitation: it is blocking (`get()`). You can't say "When this future finishes, *then* do this."

`CompletableFuture` (Java 8) introduces fully non-blocking, reactive, composable futures.

## 1. Asynchronous Pipelines
You can chain actions.

```java
CompletableFuture.supplyAsync(() -> fetchOrder(id)) // Run in common pool
    .thenApply(order -> calculateTax(order))        // Transform (map)
    .thenAccept(tax -> sendEmail(tax));             // Consume (forEach)
```
*   The main thread does not block. The pipeline executes asynchronously.

## 2. Composition
Combining multiple futures.

*   `thenCompose`: "flatMap". The result of the first future is used to launch a *second* async future.
*   `thenCombine`: "Wait for A and B, then combine results."

```java
CompletableFuture<User> userFuture = ...
CompletableFuture<Order> orderFuture = ...

userFuture.thenCombine(orderFuture, (user, order) -> {
    return new Receipt(user, order);
});
```

## 3. Exception Handling
Handles exceptions functionally.

```java
future
    .exceptionally(ex -> {
        log.error(ex);
        return defaultVal; // Recover
    });
```