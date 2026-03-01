# Custom Exceptions: Tailoring Error Handling to Your Domain

While Java provides a rich set of built-in exception classes, there are many situations where these generic exceptions do not adequately describe specific error conditions unique to your application's business logic. Creating **custom exceptions** allows you to define domain-specific errors, leading to more readable, maintainable, and robust code.

## 1. When to Create Custom Exceptions: The Rationale

Custom exceptions serve several important purposes in software design:
*   **Domain Specificity:** To represent errors that are meaningful within your application's problem domain (e.g., `InsufficientFundsException`, `ProductNotFoundException`, `InvalidUserCredentialsException`). This makes error messages more intuitive for developers.
*   **API Clarity:** When designing libraries or modules, custom exceptions explicitly communicate specific error types to API consumers, allowing them to handle those errors with precision.
*   **Layer Separation:** Custom exceptions can abstract away lower-level implementation details. For instance, a `DatabaseConnectionException` might internally wrap an `SQLException` but present a cleaner interface to the business logic layer.
*   **Enhanced Information:** Custom exceptions can include additional fields (e.g., `requiredAmount`, `errorCode`) to provide more context and data about the error, aiding in debugging and recovery.

## 2. Creating a Custom Exception: The Inheritance Choice

Creating a custom exception is straightforward: you simply extend an existing `Exception` class. The choice of which superclass to extend is critical, as it determines whether your custom exception will be **checked** or **unchecked**.

### 2.1 Custom Checked Exception (Extending `Exception`)
*   **Superclass:** Extend `java.lang.Exception` directly, or a subclass of `Exception` (but **not** `RuntimeException`).
*   **Purpose:** To signal a problem that is **predictable and potentially recoverable** by the caller. These are often external issues (e.g., file not found, network unavailable, invalid configuration). The compiler forces callers to deal with them.
*   **Usage:** If a method can throw this custom checked exception, it *must* declare it in its `throws` clause, and callers *must* either `catch` it or `throw` it themselves.

```java
// Custom Checked Exception Example
class InsufficientFundsException extends Exception { // Extends java.lang.Exception
    private double requiredAmount;
    private double availableBalance;

    // Standard constructors for exceptions
    public InsufficientFundsException(String message) {
        super(message);
    }

    public InsufficientFundsException(String message, double requiredAmount, double availableBalance) {
        super(message); // Pass message to superclass constructor
        this.requiredAmount = requiredAmount;
        this.availableBalance = availableBalance;
    }

    public InsufficientFundsException(String message, Throwable cause) {
        super(message, cause); // For exception chaining
    }

    // Add custom getters for specific error information
    public double getRequiredAmount() { return requiredAmount; }
    public double getAvailableBalance() { return availableBalance; }
}

public class BankAccount {
    private double balance;

    public BankAccount(double initialBalance) {
        if (initialBalance < 0) {
            throw new IllegalArgumentException("Initial balance cannot be negative.");
        }
        this.balance = initialBalance;
    }

    // Method declares the custom checked exception
    public void withdraw(double amount) throws InsufficientFundsException {
        if (amount < 0) {
            throw new IllegalArgumentException("Withdrawal amount cannot be negative.");
        }
        if (amount > balance) {
            // Throw the custom checked exception with relevant data
            throw new InsufficientFundsException(
                "Insufficient funds for withdrawal. Transaction failed.", amount, balance
            );
        }
        this.balance -= amount;
        System.out.println("Withdrawal successful. New balance: " + balance);
    }

    public static void main(String[] args) {
        BankAccount account = new BankAccount(500); // Initial balance
        try {
            account.withdraw(700); // This call requires handling InsufficientFundsException
        } catch (InsufficientFundsException e) {
            System.err.println("\n--- Transaction Alert ---");
            System.err.println("Problem: " + e.getMessage());
            System.err.println("Details: Required " + e.getRequiredAmount() + ", Available " + e.getAvailableBalance());
            // Log the full stack trace for debugging
            e.printStackTrace();
            System.err.println("--- End Alert ---");
        } catch (IllegalArgumentException e) {
             System.err.println("Input Error: " + e.getMessage());
        }
        System.out.println("Program continues, current balance: " + account.balance);
    }
}
```

### 2.2 Custom Unchecked Exception (Extending `RuntimeException`)
*   **Superclass:** Extend `java.lang.RuntimeException` (or any of its subclasses).
*   **Purpose:** To signal a **programming error** or a fundamental flaw in the application's logic that should typically be fixed in the code. These are generally unrecoverable runtime errors.
*   **Usage:** The compiler does **not** force callers to handle this exception. If left unhandled, it will propagate up the call stack and likely terminate the thread/application.

```java
// Custom Unchecked Exception Example
class InvalidInputDataException extends RuntimeException { // Extends RuntimeException
    public InvalidInputDataException(String message) {
        super(message);
    }

    public InvalidInputDataException(String message, Throwable cause) {
        super(message, cause); // For exception chaining
    }
}

public class DataProcessor {
    public int process(String data) {
        if (data == null || data.trim().isEmpty()) {
            // Throw an unchecked exception; no 'throws' clause needed
            throw new InvalidInputDataException("Input data cannot be null or empty.");
        }
        try {
            return Integer.parseInt(data);
        } catch (NumberFormatException e) {
            // Wrap the standard NumberFormatException in our custom unchecked exception
            throw new InvalidInputDataException("Data is not a valid number: '" + data + "'.", e);
        }
    }

    public static void main(String[] args) {
        DataProcessor processor = new DataProcessor();
        try {
            int result = processor.process("123");
            System.out.println("Result for '123': " + result); // Output: Result for '123': 123

            result = processor.process("abc"); // This will throw InvalidInputDataException
            System.out.println("Result for 'abc': " + result); 
        } catch (InvalidInputDataException e) { // Catching is optional, but good for reporting/logging
            System.err.println("Processing error: " + e.getMessage());
            if (e.getCause() != null) { // Accessing the original cause
                System.err.println("Root cause: " + e.getCause().getClass().getSimpleName() + " - " + e.getCause().getMessage());
            }
            e.printStackTrace();
        }
        System.out.println("Program continues after example block.");
    }
}
```

## 3. Standard Constructor Patterns for Custom Exceptions

It's good practice to provide at least the following four standard constructors for your custom exception classes, as they align with `Throwable`'s constructors and facilitate common usage patterns like exception chaining.

1.  `MyException(String message)`: Provides a descriptive message about the error.
2.  `MyException(String message, Throwable cause)`: Used for **exception chaining**, allowing you to wrap a lower-level exception (the `cause`) into your custom exception while preserving the original problem's context.
3.  `MyException(Throwable cause)`: For wrapping another exception when no custom message is needed (inherits message from `cause`).
4.  `MyException()`: No-argument constructor (less common for custom exceptions, as a message is usually helpful).

## 4. Summary: The Power of Domain-Specific Errors

Custom exceptions are a powerful tool for enhancing the clarity, robustness, and maintainability of your Java applications. By precisely defining error conditions within your domain, you make your APIs more understandable, simplify error handling for consumers, and improve the overall resilience of your software. The choice between a checked and unchecked custom exception should be a deliberate design decision, reflecting whether the error is a predictable, recoverable condition or an indication of a programming bug.

---

### Links to Topics:
*   [Exception Hierarchy](01_exception_hierarchy.md)
*   [try-catch-finally](02_try_catch_finally.md)
*   [try-with-resources](03_try_with_resources.md)
*   [Custom Exceptions](04_custom_exceptions.md)
*   [Best Practices](05_best_practices.md)
