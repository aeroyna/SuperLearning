# Stream Basics: The Pipeline Architecture

A `Stream` is an implementation of the **Pipeline Design Pattern**. It consists of a source, zero or more intermediate operations, and a terminal operation.

## 1. Lazy Evaluation
This is the most critical concept. When you call `filter()` or `map()`, **nothing happens**.
*   The Stream constructs a "recipe" or description of what should happen.
*   Data is only pulled from the source when the **Terminal Operation** starts running.
*   **Benefit:** Efficiency.
    *   `list.stream().filter(x -> expensive(x)).findFirst()`
    *   If the first element matches, `expensive(x)` is only called once. The rest of the list is ignored. This is "Short-Circuiting".

## 2. Stream Characteristics
*   **Not a Data Structure:** It holds no data. It pulls data.
*   **Consumable:** You can only traverse a stream once. If you need to traverse again, you must create a new stream from the source.
*   **Internal Iteration:** You don't manage the loop variable. The library manages the iteration, allowing it to optimize (e.g., parallelize) behind the scenes.

## 3. Creating Streams
*   **From Arrays:** `Arrays.stream(arr)` is efficient (spliterator knows exact size).
*   **From Files:** `Files.lines(path)` creates a stream of lines lazily. It does *not* read the whole file into memory. Crucial for large files.
*   **From Random:** `Stream.generate()` creates an infinite stream. You *must* use `limit()` to stop it.
