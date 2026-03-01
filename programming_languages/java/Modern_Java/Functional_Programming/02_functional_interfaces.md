# Functional Interfaces: The Contract

A Functional Interface is defined by a specific structural constraint: it must have **exactly one abstract method**. This constraint allows the compiler to map a Lambda Expression (which is essentially a function body) to that single method signature.

## 1. The `@FunctionalInterface` Annotation
While optional, using this annotation is a best practice. It triggers a compiler check ensuring you haven't accidentally added a second abstract method, which would break all lambdas using that interface.

## 2. The Core Four (java.util.function) Analysis

### 2.1 `Predicate<T>` (`T -> boolean`)
*   **Use Case:** Filtering, testing logic.
*   **Methods:** `test()`, `and()`, `or()`, `negate()`.
*   **Internals:** Can be chained for complex logic.
    ```java
    Predicate<String> valid = s -> s != null;
    Predicate<String> longEnough = s -> s.length() > 5;
    // Chaining creates a composite predicate
    Predicate<String> combined = valid.and(longEnough); 
    ```

### 2.2 `Function<T, R>` (`T -> R`)
*   **Use Case:** Transformation, mapping.
*   **Methods:** `apply()`, `andThen()`, `compose()`.
*   **Identity:** `Function.identity()` returns a function that returns its input. Useful for collectors (e.g., `groupingBy(Function.identity())`).

### 2.3 `Consumer<T>` (`T -> void`)
*   **Use Case:** Side-effects (printing, saving to DB, updating UI).
*   **Methods:** `accept()`, `andThen()`.
*   **Chaining:** `c1.andThen(c2)` runs c1, then c2.

### 2.4 `Supplier<T>` (`() -> T`)
*   **Use Case:** Lazy evaluation, factory patterns.
*   **Importance:** `Optional.orElseGet(Supplier)` vs `Optional.orElse(T)`.
    *   `orElse(new BigObject())`: `new BigObject()` is created *even if* the optional is present (wasteful).
    *   `orElseGet(() -> new BigObject())`: The lambda is only executed (and object created) *if* value is missing.

## 3. Primitive Specializations (Performance)
Generic types (`Function<Integer, Integer>`) involve **Boxing** (int -> Integer) and **Unboxing** (Integer -> int). This kills performance in numerical computing.
*   **Use:** `IntFunction`, `DoubleConsumer`, `LongPredicate`.
*   **Benefit:** Operates directly on stack primitives. No heap allocation for wrapper objects.
