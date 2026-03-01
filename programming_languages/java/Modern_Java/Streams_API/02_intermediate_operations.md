# Intermediate Operations: Building the Recipe

Intermediate operations transform a stream into another stream. They are categorized as **Stateless** or **Stateful**.

## 1. Stateless Operations
These operations process elements independently. They don't need to know about other elements.
*   **`filter(Predicate)`**: Passes element if predicate is true.
*   **`map(Function)`**: Transforms element.
*   **`flatMap(Function)`**: Transforms 1 element into N elements (0..infinity). Flattens the structure.
*   **Performance:** Highly parallelizable. No synchronization needed between threads processing different elements.

## 2. Stateful Operations
These operations need to see *all* (or some history of) elements to make a decision. They break the "independent flow".
*   **`sorted()`**: Must see the *entire* stream to determine the first element. This is a **barrier**. It kills parallelism benefits for infinite streams (actually, it crashes) and requires buffering all data.
*   **`distinct()`**: Needs to maintain a Set of seen elements.
*   **`limit(n)` / `skip(n)`**: Needs to count elements. Ordering matters.

## 3. Short-Circuiting Operations
Operations that can cut the processing short.
*   `limit()`: Can stop pulling from upstream once limit is reached.
*   Combined with `findFirst()`, this makes searching massive datasets efficient.
