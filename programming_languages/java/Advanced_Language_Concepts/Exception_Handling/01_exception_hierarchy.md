# Exception Hierarchy

In Java, errors and exceptions are events that disrupt the normal flow of a program. The Java Exception Hierarchy is a structured way to manage these events.

## 1. The `Throwable` Class

All error and exception classes in Java inherit from the `java.lang.Throwable` class. `Throwable` has two direct subclasses:
1.  `Error`
2.  `Exception`

```
      java.lang.Object
             |
      java.lang.Throwable
        /           \
    Error        Exception
                   /       \
         Checked Exception   RuntimeException
```

---

## 2. `Error`

*   **Definition:** Errors are abnormal conditions that indicate serious problems that an application should not try to catch. They are usually caused by situations beyond the control of the application.
*   **Examples:** `OutOfMemoryError`, `StackOverflowError`, `VirtualMachineError`.
*   **Handling:** You generally **do not** catch `Error`s. They indicate a problem with the JVM itself or the system resources, and typically lead to application termination.

---

## 3. `Exception`

*   **Definition:** Exceptions are abnormal conditions that an application **should anticipate and try to handle**. They represent situations that occur due to program logic or external factors that can often be resolved or managed.
*   **Examples:** `IOException`, `SQLException`, `FileNotFoundException`, `NullPointerException`, `ArrayIndexOutOfBoundsException`.
*   `Exception` itself has two main types: **Checked Exceptions** and **Unchecked Exceptions**.

---

## 4. Checked vs. Unchecked Exceptions

This is a crucial distinction in Java exception handling.

### 4.1 Checked Exceptions
*   **Definition:** Exceptions that are **checked at compile time**. The compiler forces you to either handle them (with a `try-catch` block) or declare that your method might throw them (with a `throws` clause).
*   **Inheritance:** All exceptions that directly inherit from `Exception` (but not `RuntimeException`) are checked exceptions.
*   **Purpose:** Used for predictable but unpreventable problems. These are often external issues that your program can recover from (e.g., file not found, network issues).
*   **Examples:** `IOException`, `SQLException`, `FileNotFoundException`, `ClassNotFoundException`.

```java
// Example: Checked Exception
import java.io.FileReader;
import java.io.IOException;

public class MyClass {
    public static void readFile(String fileName) throws IOException { // Declared
        FileReader file = new FileReader(fileName);
        // ... read from file ...
        file.close(); // Need to handle IOException here too
    }

    public static void main(String[] args) {
        try {
            readFile("myfile.txt"); // Must be caught
        } catch (IOException e) {
            System.out.println("Error reading file: " + e.getMessage());
        }
    }
}
```

### 4.2 Unchecked Exceptions (Runtime Exceptions)
*   **Definition:** Exceptions that are **not checked at compile time**. The compiler does not force you to handle or declare them.
*   **Inheritance:** All exceptions that inherit from `java.lang.RuntimeException` are unchecked exceptions.
*   **Purpose:** Typically indicate programming bugs. These are often preventable problems that should ideally be fixed in the code (e.g., incorrect array indexing, null pointers).
*   **Examples:** `NullPointerException`, `ArrayIndexOutOfBoundsException`, `IllegalArgumentException`, `ArithmeticException`.
*   **Handling:** You can catch them, but generally, the best practice is to fix the underlying bug rather than catching them and letting the program continue in a potentially unstable state.

```java
// Example: Unchecked Exception
public class UncheckedExample {
    public static void main(String[] args) {
        // NullPointerException - a common programming bug
        String str = null;
        // System.out.println(str.length()); // This line would throw NPE at runtime

        // ArrayIndexOutOfBoundsException - another common programming bug
        int[] numbers = new int[5];
        // System.out.println(numbers[10]); // This line would throw AIOOBE at runtime
    }
}
```

---

## 5. Summary of Exception Types

| Type           | Inherits from  | Checked by Compiler | Best Practice                           |
| :------------- | :------------- | :------------------ | :-------------------------------------- |
| **Error**      | `Throwable`    | No                  | Do not catch, fix system issues         |
| **Checked**    | `Exception`    | Yes                 | Catch or Declare (`throws`), recover    |
| **Unchecked**  | `RuntimeException` (extends `Exception`) | No | Fix the bug, prevent occurrence |

Understanding this hierarchy is crucial for writing robust and maintainable Java code that effectively handles unexpected situations.