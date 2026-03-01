# Optional Basics: Design Intent

`Optional` was designed primarily as a **return type** for methods where "no result" is a valid possibility. It forces the client to handle the absence of a value.

## 1. The Cost of Optional
*   **Object Allocation:** `Optional` is an object. Wrapping a value in `Optional` creates a new heap object.
*   **Indirection:** Accessing the value requires an extra pointer dereference.
*   **Usage Rule:** Do NOT use `Optional` in performance-critical inner loops.

## 2. Anti-Patterns (What NOT to do)
*   **As Field:** `Optional` is **not Serializable**. If you use it as a field in a class that might be serialized, you will get a `NotSerializableException`.
    ```java
    class User {
        Optional<String> address; // BAD! Not serializable.
    }
    ```
*   **As Parameter:** `public void foo(Optional<String> s)`.
    *   This forces the caller to wrap variables: `foo(Optional.of("hello"))`.
    *   It doesn't solve the null problem because `s` itself can be `null`!
    *   *Fix:* Just use `String s` and allow/check for null, or use `@Nullable` annotations.

## 3. Correct Usage
Use it strictly as a return type for libraries/APIs.
```java
public Optional<User> findUser(String id) { ... }
```
