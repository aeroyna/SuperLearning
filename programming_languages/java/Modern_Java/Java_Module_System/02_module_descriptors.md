# Module Descriptors and Directives

The `module-info.java` is compiled into `module-info.class`. It defines the module's contract.

## 1. Readability vs. Accessibility
*   **Readability:** Module A "reads" Module B if A `requires` B.
*   **Accessibility:** A class in A can access a class in B only if:
    1.  A reads B.
    2.  B `exports` the package containing the class.
    3.  The class is `public`.

## 2. Advanced Directives

### `opens` vs `exports`
*   **`exports`**: Allows *compile-time* and *runtime* access. But `private` members remain private.
*   **`opens`**: Allows *runtime-only* access via **Reflection**. Deep reflection (setAccessible) is allowed. This is crucial for frameworks like Spring/Hibernate.

```java
module com.my.app {
    // Allow Hibernate to reflect on entities
    opens com.my.app.model to org.hibernate.orm.core;
}
```

### `provides` ... `with` (Service Loading)
JPMS integrates natively with `ServiceLoader`.
*   **Interface:** `module A { exports com.api; }`
*   **Impl:** `module B { requires A; provides com.api.MyService with com.impl.MyServiceImpl; }`
*   **Consumer:** `module C { requires A; uses com.api.MyService; }`

## 3. The "Split Package" Problem
JPMS forbids "Split Packages": Two modules cannot export the *same* package name.
*   *Legacy:* Often jars had overlapping packages (e.g., `javax.annotation`).
*   *Fix:* You cannot put both on the module path. You must patch them or use one.
