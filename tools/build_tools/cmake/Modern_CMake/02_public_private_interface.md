# Public, Private, & Interface 🔒

Understanding scope keywords is key to mastering Modern CMake. They control **Usage Requirements propagation**.

---

## 1. The Keywords

When modifying a target (e.g., `target_link_libraries` or `target_include_directories`), you must specify scope.

| Keyword | Usage for Self | Usage for Consumers |
| :--- | :--- | :--- |
| **PRIVATE** | ✅ Used to build this target | ❌ NOT passed to consumers |
| **INTERFACE** | ❌ NOT used to build this target | ✅ Passed to consumers |
| **PUBLIC** | ✅ Used to build this target | ✅ Passed to consumers |

---

## 2. Real World Example

Imagine a library `Car` that uses an engine library `Engine`.

### Scenario A: Private Dependency
The `Car` uses the `Engine` internally. Users of `Car` don't need to know `Engine` exists.
```cmake
add_library(Car car.cpp)
target_include_directories(Car PUBLIC include)

# Only Car needs Engine headers to compile.
# Users of Car do NOT need Engine headers.
target_link_libraries(Car PRIVATE Engine)
```

### Scenario B: Public Dependency
The `Car` exposes `Engine` types in its public API (e.g., `Car::getEngine()`).
```cmake
add_library(Car car.cpp)
target_link_libraries(Car PUBLIC Engine)
```
*   Now, if `my_app` links to `Car`, it **automatically** gets linked to `Engine` and adds `Engine`'s headers to its path.

### Scenario C: Interface (Header-Only Lib)
A header-only library has no source files to compile, but provides headers to consumers.
```cmake
add_library(MyHeaderLib INTERFACE)
target_include_directories(MyHeaderLib INTERFACE include)
```
