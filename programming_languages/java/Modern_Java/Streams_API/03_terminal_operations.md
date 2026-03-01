# Terminal Operations: Triggering Execution

Terminal operations are the finish line. They trigger the lazy execution of the pipeline and produce a non-stream result.

## 1. Side-Effects vs. Reduction

### `forEach` (Side-Effect)
*   `stream.forEach(System.out::println)`
*   **Use case:** Printing, interacting with external world.
*   **Concurrency Warning:** `forEach` does not guarantee order in parallel streams. Use `forEachOrdered` if order matters (but this kills parallelism benefits). Avoid modifying shared state inside `forEach`.

### `collect` (Mutable Reduction)
*   Accumulates elements into a container (List, Map).
*   **Collectors:** The `Collectors` utility class provides factories.
    *   `toList()`, `toSet()`
    *   `joining()`: Concatenates strings.
    *   `groupingBy()`: Analogous to SQL GROUP BY. Returns `Map<Key, List<Items>>`.
    *   `partitioningBy()`: Groups into `true` and `false` lists based on predicate.

### `reduce` (Immutable Reduction)
*   Combines elements into a single value using an accumulator function.
*   `T reduce(T identity, BinaryOperator<T> accumulator)`
*   *Example:* Summing numbers. `reduce(0, (a, b) -> a + b)`.
*   **Concept:** This is the "Reduce" in MapReduce. Ideally suited for parallel computation because it is associative.

## 2. Short-Circuiting Terminals
*   `findFirst()`, `findAny()`
*   `anyMatch()`, `allMatch()`, `noneMatch()`
*   These stop processing as soon as the result is determined.
