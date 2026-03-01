# Functional Handling of Absence

The true power of `Optional` is not `get()` or `isPresent()`, but the functional pipeline it enables.

## 1. Monadic Operations
`Optional` behaves like a Monad (a container representing a context).
*   **`map`**: "If a value is here, modify it."
*   **`flatMap`**: "If a value is here, use it to compute another Optional."

## 2. Lazy Evaluation of Defaults
*   **`orElse(T)`**: Eager. The argument is evaluated *before* checking if Optional is empty.
    ```java
    // "Expensive" is calculated even if value is present!
    opt.orElse(calculateExpensiveDefault()); 
    ```
*   **`orElseGet(Supplier)`**: Lazy. The Supplier is only called if Optional is empty.
    ```java
    // Safe and efficient
    opt.orElseGet(() -> calculateExpensiveDefault());
    ```

## 3. Java 9+ Improvements
*   **`ifPresentOrElse(Consumer, Runnable)`**: Do this if present, else do that.
    ```java
    opt.ifPresentOrElse(
        val -> print(val),
        () -> print("Not Found")
    );
    ```
*   **`stream()`**: Converts `Optional` to a `Stream` of 0 or 1 element. Great for flatMapping a stream of optionals.
    ```java
    List<Optional<String>> list = ...;
    list.stream()
        .flatMap(Optional::stream) // Removes empties, unwraps values
        .collect(toList());
    ```
