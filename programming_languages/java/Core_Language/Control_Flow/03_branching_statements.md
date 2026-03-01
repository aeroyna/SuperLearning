# Branching Statements

Branching statements allow you to alter the flow of execution within loops or methods, enabling you to jump to a different part of your code.

## 1. `break` Statement

The `break` statement is used to terminate the loop or `switch` statement immediately. Control resumes at the statement immediately following the terminated loop or `switch`.

### `break` in `switch`
(Covered in Conditional Statements). It prevents fall-through.

### `break` in Loops
```java
for (int i = 0; i < 10; i++) {
    if (i == 5) {
        break; // Exits the for loop when i is 5
    }
    System.out.println("i: " + i);
}
// Output: 0, 1, 2, 3, 4
// Loop terminates when i becomes 5.
```

### Labeled `break` (Rarely Used)
You can use `break` to exit an outer loop from within an inner loop. This is done by specifying a label.
```java
outerLoop: // Label
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        if (i == 1 && j == 1) {
            break outerLoop; // Exits both loops
        }
        System.out.println("i: " + i + ", j: " + j);
    }
}
// Output:
// i: 0, j: 0
// i: 0, j: 1
// i: 0, j: 2
// i: 1, j: 0
// Loop terminates when i is 1 and j is 1.
```
*   Labeled `break` can make code harder to read and debug. Often, redesigning the loop or using a `boolean` flag is a cleaner alternative.

---

## 2. `continue` Statement

The `continue` statement skips the rest of the current iteration of a loop and proceeds to the next iteration.

### `continue` in Loops
```java
for (int i = 0; i < 5; i++) {
    if (i == 2) {
        continue; // Skips the rest of this iteration when i is 2
    }
    System.out.println("i: " + i);
}
// Output: 0, 1, 3, 4
// When i is 2, the print statement is skipped, and loop continues with i=3.
```

### Labeled `continue` (Rarely Used)
Used to skip the current iteration of an outer loop from within an inner loop.
```java
outerLoop:
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        if (i == 1) {
            continue outerLoop; // Skips to next iteration of outerLoop when i is 1
        }
        System.out.println("i: " + i + ", j: " + j);
    }
}
// Output:
// i: 0, j: 0
// i: 0, j: 1
// i: 0, j: 2
// i: 2, j: 0
// i: 2, j: 1
// i: 2, j: 2
// When i is 1, the inner loop for i=1 is entirely skipped.
```

---

<h2>3. `return` Statement</h2>
The `return` statement is used to exit from a method. It can optionally return a value if the method has a non-`void` return type.

### `return` from `void` method
```java
public void printMessage(String message) {
    if (message == null || message.isEmpty()) {
        return; // Exits the method
    }
    System.out.println(message);
}
```

### `return` from method with value
```java
public int add(int a, int b) {
    int sum = a + b;
    return sum; // Returns the calculated sum
}

// Can be simplified:
public int addSimple(int a, int b) {
    return a + b;
}
```
*   When `return` is executed, the method's execution stops immediately, and control is passed back to the caller.
*   Any code after `return` within the same method path will be unreachable and cause a compile-time error.