# Method References: Cleaner Lambdas

Method references (`::`) are strictly syntactic sugar for specific types of lambda expressions. However, understanding *which* type you are using is vital for readability.

## 1. The Four Types: Deep Dive

### 1.1 Static Method (`ClassName::staticMethod`)
*   **Lambda:** `(args) -> ClassName.staticMethod(args)`
*   **Analogy:** A function pointer to a global function.
*   **Example:** `Integer::parseInt` maps to `s -> Integer.parseInt(s)`.

### 1.2 Instance Method of Particular Object (`instance::method`)
*   **Lambda:** `(args) -> instance.method(args)`
*   **Mechanism:** The specific object `instance` is **captured** by the lambda. This works like a "Capturing Lambda".
*   **Cost:** Creates a new object allocation to hold the reference to `instance`.
*   **Example:** `System.out::println`. Here, `System.out` (the PrintStream object) is captured.

### 1.3 Instance Method of Arbitrary Object (`ClassName::method`)
*   **Lambda:** `(arg0, rest) -> arg0.method(rest)`
*   **Mechanism:** The **first argument** of the lambda becomes the target (`this`) of the method call.
*   **Example:** `String::toLowerCase`.
    *   Signature: `Function<String, String>`.
    *   Input: A String `s`.
    *   Execution: `s.toLowerCase()`.

### 1.4 Constructor (`ClassName::new`)
*   **Lambda:** `(args) -> new ClassName(args)`
*   **Usage:** Factories.
    ```java
    Supplier<List<String>> s = ArrayList::new;
    Function<Integer, List<String>> f = ArrayList::new; // Calls new ArrayList(initialCapacity)
    ```
*   **Array Constructor:** `String[]::new` maps to `size -> new String[size]`. Essential for `stream.toArray(String[]::new)`.
