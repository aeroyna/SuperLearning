# Loading, Linking, and Initialization

The lifecycle of a class in the JVM.

## 1. Loading
Fetching the binary `.class` data and creating a `Class` object in the Heap.

## 2. Linking
1.  **Verification:** Checks byte stream structure, stack consistency, and type safety. Ensuring the code won't crash the JVM.
2.  **Preparation:** Allocates memory for static variables and sets default values (0, null).
3.  **Resolution:** Replaces symbolic references ("com.example.User") with direct memory addresses.

## 3. Initialization (The `<clinit>` method)
This is when the class actually "starts".
*   Executes `static { ... }` blocks.
*   Assigns actual values to static variables (`static int x = 5;`).

**Triggers for Initialization:**
*   `new` instance created.
*   Accessing a static field (except `static final` constants).
*   Calling a static method.
*   Reflection (`Class.forName()`).
*   `main()` method class.

*Note:* Loading a class does *not* necessarily trigger Initialization. `Class.forName("MyClass", false, loader)` loads but doesn't initialize.