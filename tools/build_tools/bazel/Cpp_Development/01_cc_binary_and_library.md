# cc_binary & cc_library 🧱

These are the two workhorse rules for C++ development in Bazel.

---

## 1. `cc_binary`

Compiles C++ sources into an executable program.

```python
cc_binary(
    name = "hello_world",
    srcs = ["main.cc", "helper.cc", "helper.h"],
    deps = [":my_lib"],
    copts = ["-O3", "-Wall"],
)
```

*   **srcs**: Source files (`.cc`, `.cpp`, `.c`) AND private headers (`.h`).
*   **deps**: Libraries to link against.
*   **copts**: Compiler flags passed to GCC/Clang (e.g., optimization).
*   **linkopts**: Linker flags (e.g., `-lm`, `-lpthread`).

### Running the Binary
```bash
bazel run //path/to:hello_world
```

---

## 2. `cc_library`

Compiles code into a static (`.a`) or shared (`.so` / `.dylib`) library.

```python
cc_library(
    name = "math_utils",
    hdrs = ["math_utils.h"],   # Public Headers
    srcs = ["math_utils.cc"],  # Implementation
    deps = [],
    visibility = ["//visibility:public"],
)
```

### The Difference between `srcs` and `hdrs` 🚨
*   **`hdrs` (Public)**: Files listed here are exposed to consumers. If Target A depends on Target B, Target A can `#include` the files in B's `hdrs`.
*   **`srcs` (Private)**: Implementation files and *private* headers. If you put a `.h` file here, it is hidden. Consumers cannot include it.

> **Rule of Thumb**: If it's in the public interface, put it in `hdrs`. If it's an internal helper, put it in `srcs`.

---

## 3. `cc_test`

Compiles a test executable. Commonly used with GoogleTest.

```python
cc_test(
    name = "math_test",
    srcs = ["math_test.cc"],
    deps = [
        ":math_utils",
        "@com_google_googletest//:gtest_main",
    ],
)
```

### Running Tests
```bash
bazel test //path/to:math_test
```
