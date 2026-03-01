# Loops: Orchestrating Repetitive Tasks

Loops are fundamental control flow statements that enable your program to repeatedly execute a block of code. They are essential for tasks like processing lists of data, repeating calculations, or waiting for specific conditions. Java offers several types of loops, each suited for different scenarios.

## 1. The `for` Loop: Iteration with Known Bounds

The `for` loop is typically used when you know, or can calculate, the number of iterations in advance. It provides a compact syntax for managing a loop counter.

### Basic Syntax
```java
for (initialization; termination; increment/decrement) {
    // code to be executed repeatedly
}
```
*   **`initialization`:** Executed exactly once at the beginning of the loop. Often used to declare and initialize a loop control variable (e.g., `int i = 0;`).
*   **`termination`:** A boolean expression evaluated before each iteration. If it evaluates to `false`, the loop terminates.
*   **`increment/decrement`:** Executed after each iteration of the loop body. Often used to update the loop control variable (e.g., `i++`, `i = i + 2`).

### Example
```java
for (int i = 0; i < 5; i++) {
    System.out.println("For Loop Iteration: " + i);
}
// Output: Iteration 0, 1, 2, 3, 4
```

### Variations and Nuances
*   **Optional Parts:** All three parts of the `for` loop header are optional, but the semicolons must remain.
    *   `for (;;)`: This creates an **infinite loop**, equivalent to `while (true)`.
    *   `for (int i = 0; i < 5; ) { System.out.println(i); i++; }`: Increment moved to body.
*   **Multiple Variables:** You can declare and initialize multiple variables (of the same type) in the initialization part and include multiple increment/decrement statements.
    ```java
    for (int i = 0, j = 10; i < j; i++, j--) {
        System.out.println("i: " + i + ", j: " + j);
    }
    ```
*   **Scope:** Variables declared in the `initialization` part are local to the `for` loop and cannot be accessed outside it.

## 2. The Enhanced `for-each` Loop (for Iterating Collections)

Introduced in Java 5, the enhanced `for-each` loop provides a simpler, more readable way to iterate over arrays and objects that implement the `Iterable` interface (like all Java Collections).

### Syntax
```java
for (Type element : iterable) {
    // code to be executed for each element
}
```
*   **`Type`:** The data type of the elements in the `iterable`.
*   **`element`:** A temporary variable that holds the current element in each iteration.
*   **`iterable`:** An array (e.g., `int[]`) or an object that implements `java.lang.Iterable` (e.g., `ArrayList<String>`).

### Example
```java
int[] numbers = {1, 2, 3, 4, 5};
for (int num : numbers) { // 'num' automatically gets each element from 'numbers'
    System.out.println("Number: " + num);
}

List<String> names = new ArrayList<>(Arrays.asList("Alice", "Bob", "Charlie"));
for (String name : names) {
    System.out.println("Name: " + name);
}
```
*   **Internal Mechanism:** The `for-each` loop internally uses an `Iterator` for collections and a counter for arrays.

### Limitations and Nuances
1.  **Read-Only Access:** The `for-each` loop provides read-only access to the elements. You cannot modify the elements themselves by assigning a new value to the `element` variable within the loop (e.g., `name = name.toUpperCase();` inside the loop will not change the actual element in the `names` list).
2.  **No Index Access:** You cannot directly access the index of the current element within a `for-each` loop. If you need the index, you must use a traditional `for` loop.
3.  **No Element Removal:** You cannot safely remove elements from a collection during a `for-each` loop. Doing so will typically result in a `ConcurrentModificationException`. To remove elements while iterating, you must use an explicit `Iterator` and its `remove()` method.

## 3. The `while` Loop: Iteration with Unknown Bounds

The `while` loop executes a block of code repeatedly as long as a specified condition remains `true`. It's best used when the number of iterations is unknown and depends on runtime conditions.

### Syntax
```java
while (condition) {
    // code to be executed repeatedly
}
```
*   **`condition`:** A boolean expression that is evaluated *before* each iteration. If `condition` is `false` initially, the loop body will never execute.

### Example
```java
int count = 0;
while (count < 3) { // Condition checked before each iteration
    System.out.println("While Loop Count: " + count);
    count++; // Crucial: must update a variable in the condition or risk infinite loop
}
// Output: Count: 0, 1, 2
```

## 4. The `do-while` Loop: Guaranteed First Execution

Similar to the `while` loop, but with a key difference: the code block is guaranteed to execute at least once, because the condition is checked *after* the first iteration.

### Syntax
```java
do {
    // code to be executed at least once
} while (condition); // Note the semicolon after the while condition
```

### Example
```java
int input;
Scanner scanner = new Scanner(System.in); // Used for example, usually use try-with-resources

do {
    System.out.print("Enter a number (0 to exit): ");
    input = scanner.nextInt();
    System.out.println("You entered: " + input);
} while (input != 0); // Condition checked after loop body executes
scanner.close();
// Loop will run at least once, even if the user enters 0 initially.
```
*   **Nuance:** The semicolon after the `while (condition)` is mandatory for `do-while` loops.

## 5. Infinite Loops: Intentional vs. Accidental

An infinite loop is a loop that never terminates because its termination condition is never met.

### Intentional Infinite Loops
Sometimes, infinite loops are designed for specific purposes, such as:
*   **Event Loops:** In GUI applications or server applications, a main thread might run an infinite loop waiting for events or client connections.
*   **Game Loops:** Game engines often have a `while (true)` loop that continuously updates game state and renders frames.
*   **Daemon Threads:** Background threads that continuously monitor a resource.

```java
// Server listening loop
while (true) {
    Socket clientSocket = serverSocket.accept(); // Blocks until a client connects
    // Handle clientSocket
}
```

### Accidental Infinite Loops (Bugs!)
These occur when the loop's termination condition is inadvertently never met, leading to:
*   **Program Hang:** The application becomes unresponsive.
*   **Resource Exhaustion:** Excessive CPU usage, memory leaks (if objects are continuously created), leading to application crashes.

```java
int i = 0;
while (i < 5) {
    System.out.println("Stuck in a loop!");
    // Missing i++; -> Infinite loop!
}
```

## 6. Performance Considerations
*   For primitive arrays, a traditional `for` loop might offer a slight performance edge due to direct index access and better cache utilization, especially in performance-critical code.
*   For `ArrayList`s, the `for` loop and `for-each` loop are often equally optimized by the JIT compiler.
*   For `LinkedList`s, `for-each` (which uses an Iterator) is generally more efficient than a traditional `for` loop with `get(index)` because `get(index)` on a `LinkedList` is `O(n)`.

Choose the loop type that best fits the logic, clearly expresses intent, and prioritizes readability, while being mindful of performance implications for very large datasets or performance-critical sections.

---

### Links to Topics:
*   [Conditional Statements](01_conditional_statements.md)
*   [Loops](02_loops.md)
*   [Branching Statements](03_branching_statements.md)
