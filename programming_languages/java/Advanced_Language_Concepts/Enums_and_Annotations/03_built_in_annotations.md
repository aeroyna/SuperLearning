# Built-in Annotations

Annotations, introduced in **Java 5**, are a form of metadata that can be added to Java code. They provide information about the code without directly affecting its execution. Annotations are processed by various tools (compiler, JVM, build tools, IDEs) to perform tasks like error checking, code generation, or runtime processing.

## 1. Types of Annotations

### A. Marker Annotations
Annotations with no elements. (e.g., `@Override`)

### B. Single-Value Annotations
Annotations with a single element, which can be abbreviated. (e.g., `@SuppressWarnings("unchecked")`)

### C. Multi-Value Annotations
Annotations with multiple elements. (e.g., `@Author(name="John Doe", date="2023-01-01")`)

---

## 2. Common Standard Annotations

Java provides several standard, built-in annotations that you will encounter frequently.

### 2.1 `@Override`
*   **Purpose:** Marks a method as overriding a method from its superclass or implementing a method from an interface.
*   **Benefit:** Compiler checks if the method actually overrides something. If the signature is incorrect, it will report a compile-time error, preventing subtle bugs.
*   **Retention:** `SOURCE` (discarded after compilation).
*   **Usage Example:**
    ```java
    class Animal {
        public void makeSound() { /* ... */ }
    }

    class Dog extends Animal {
        @Override // Compile-time check: ensures makeSound exists in Animal with this signature
        public void makeSound() {
            System.out.println("Woof!");
        }
    }
    ```

### 2.2 `@Deprecated`
*   **Purpose:** Marks a class, method, or field as deprecated, meaning it's discouraged from use, usually because it's been superseded by newer, better alternatives, or will be removed in future versions.
*   **Benefit:** Compiler generates a warning if deprecated code is used. IDEs typically strike through deprecated elements.
*   **Retention:** `RUNTIME` (available at runtime via Reflection).
*   **Usage Example:**
    ```java
    @Deprecated(since="1.5", forRemoval=true) // More detailed since Java 9
    public void oldMethod() {
        System.out.println("This method is deprecated. Use newMethod() instead.");
    }

    // Usage:
    // MyClass obj = new MyClass();
    // obj.oldMethod(); // Compiler will warn about using a deprecated method
    ```

### 2.3 `@SuppressWarnings`
*   **Purpose:** Suppresses compiler warnings for a specific annotated element (class, method, or field).
*   **Benefit:** Allows developers to acknowledge a warning but explicitly tell the compiler that it's intentionally ignored (e.g., in cases of deliberate raw type usage for compatibility reasons, or when a generic cast is known to be safe).
*   **Retention:** `SOURCE`.
*   **Usage Example:**
    ```java
    @SuppressWarnings("unchecked") // Suppress warning for unchecked cast
    public List<String> getStrings() {
        List rawList = new ArrayList();
        rawList.add("String 1");
        return rawList; // Compiler would normally warn here
    }

    @SuppressWarnings({"deprecation", "rawtypes"}) // Multiple warnings
    public void useDeprecatedStuff() {
        // ... code using deprecated methods and raw types ...
    }
    ```
*   **Caution:** Use `SuppressWarnings` sparingly and with clear justification. Suppressing too many warnings can hide legitimate problems.

### 2.4 `@FunctionalInterface` (Java 8+)
*   **Purpose:** Indicates that an interface is a "functional interface," meaning it has exactly one abstract method.
*   **Benefit:** Compiler verifies that the interface adheres to the functional interface contract. Enables the use of lambda expressions for implementing this interface.
*   **Retention:** `RUNTIME`.
*   **Usage Example:**
    ```java
    @FunctionalInterface
    interface MyFunction {
        void apply();
        // void anotherMethod(); // Compile-time error: more than one abstract method
        default void doSomething() {} // Default methods are allowed
    }
    ```

### 2.5 `@SafeVarargs` (Java 7+)
*   **Purpose:** Used on methods or constructors that take `varargs` parameters. It assures the compiler that the code will not perform unsafe operations on the `varargs` array.
*   **Benefit:** Suppresses warnings about potential heap pollution when dealing with generic `varargs`.
*   **Retention:** `RUNTIME`.
*   **Usage Example:**
    ```java
    @SafeVarargs // Assures compiler that method doesn't do unsafe things with 'items'
    public final <T> List<T> asList(T... items) {
        return Arrays.asList(items);
    }
    ```

---

## 3. Metadata and Tooling
Annotations are powerful because they allow you to embed metadata directly into your source code. This metadata can then be used by:
*   **Compilers:** (`@Override`, `@SuppressWarnings`)
*   **IDEs:** Provide warnings, code completion, suggestions.
*   **Build Tools:** Generate code, configure frameworks.
*   **JVM at Runtime:** (via Reflection) Process annotations for framework configuration (e.g., Spring's `@Autowired`, JUnit's `@Test`).

Understanding these built-in annotations is crucial for reading, writing, and debugging modern Java code effectively.