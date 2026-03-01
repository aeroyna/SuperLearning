# Loops

Loops are fundamental control flow statements that allow you to repeatedly execute a block of code.

## 1. The `for` Loop

The `for` loop is typically used when you know, or can calculate, the number of iterations in advance.

### Basic Syntax
```java
for (initialization; termination; increment/decrement) {
    // code to be executed repeatedly
}
```
*   **`initialization`:** Executed once at the beginning (e.g., `int i = 0;`).
*   **`termination`:** A boolean expression checked before each iteration. If `false`, the loop ends.
*   **`increment/decrement`:** Executed after each iteration (e.g., `i++`, `i = i + 2`).

### Example
```java
for (int i = 0; i < 5; i++) {
    System.out.println("Iteration: " + i);
}
// Output:
// Iteration: 0
// Iteration: 1
// Iteration: 2
// Iteration: 3
// Iteration: 4
```

### Variations
*   **Infinite Loop:** `for (;;)` or `while (true)`
*   **Multiple Initializations/Increments:**
    ```java
    for (int i = 0, j = 10; i < j; i++, j--) {
        System.out.println("i: " + i + ", j: " + j);
    }
    ```
*   **Optional Parts:** Any of the three parts can be omitted, but semicolons must remain.

---

## 2. The Enhanced `for-each` Loop

Introduced in Java 5, this loop provides a simpler way to iterate over arrays and collections. It's often called the "for-each" loop.

### Syntax
```java
for (Type element : iterable) {
    // code to be executed for each element
}
```
*   **`iterable`:** An array or any object that implements the `Iterable` interface (like `ArrayList`, `HashSet`).

### Example
```java
int[] numbers = {1, 2, 3, 4, 5};
for (int num : numbers) {
    System.out.println("Number: " + num);
}

// Cannot modify array elements directly with for-each loop
String[] names = {"Alice", "Bob"};
for (String name : names) {
    // name = name.toUpperCase(); // This will not modify the array 'names'
}
```
*   **Limitation:** You cannot use the for-each loop to modify the contents of the array/collection (by assignment to `element`), nor can you iterate backwards or skip elements based on an index.

---

## 3. The `while` Loop

The `while` loop executes a block of code as long as a specified condition is true. It's best used when the number of iterations is unknown and depends on runtime conditions.

### Syntax
```java
while (condition) {
    // code to be executed repeatedly
}
```
*   **`condition`:** A boolean expression checked *before* each iteration.

### Example
```java
int count = 0;
while (count < 3) {
    System.out.println("Count: " + count);
    count++;
}
// Output:
// Count: 0
// Count: 1
// Count: 2
```

---

## 4. The `do-while` Loop

Similar to the `while` loop, but guarantees that the code block is executed at least once, because the condition is checked *after* the first iteration.

### Syntax
```java
do {
    // code to be executed repeatedly
} while (condition); // Note the semicolon
```

### Example
```java
int input;
Scanner scanner = new Scanner(System.in);
do {
    System.out.print("Enter a number (0 to exit): ");
    input = scanner.nextInt();
    System.out.println("You entered: " + input);
} while (input != 0);
scanner.close();
// Loop will run at least once, even if user enters 0 initially.
```
*   **Caution:** Don't forget the semicolon after the `while` condition in a `do-while` loop.

---

## 5. Infinite Loops
A loop that never terminates because its termination condition is never met.
```java
while (true) {
    System.out.println("This will print forever!");
}

for (;;) {
    System.out.println("This will also print forever!");
}
```
*   Can be useful in specific scenarios (e.g., event loops in server applications) but must be carefully managed to avoid resource exhaustion.
*   In normal applications, they usually indicate a bug.