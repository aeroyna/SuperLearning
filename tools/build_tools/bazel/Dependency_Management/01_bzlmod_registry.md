# Dependency Management with Bzlmod 📦

**Bzlmod** is the new external dependency system introduced in Bazel 5.0 and standard in Bazel 6.0+. It replaces the legacy `WORKSPACE` file. It works similarly to `npm` (package.json) or `cargo` (Cargo.toml).

---

## 1. The Central Registry (BCR)

Bazel has a central registry called the **Bazel Central Registry (BCR)**. It hosts modules like `googletest`, `abseil-cpp`, `protobuf`, etc.

*   **URL**: [registry.bazel.build](https://registry.bazel.build)

---

## 2. `MODULE.bazel` Basics

This file lives at your workspace root.

```python
# MODULE.bazel

# 1. Define your module
module(
    name = "my_project",
    version = "1.0",
)

# 2. Add dependencies
bazel_dep(name = "googletest", version = "1.12.1")
bazel_dep(name = "abseil-cpp", version = "20230125.1")
```

---

## 3. Using Dependencies in `BUILD` Files

Once declared in `MODULE.bazel`, you can use them in your `deps` using the `@<module_name>` syntax.

```python
cc_test(
    name = "my_test",
    srcs = ["test.cc"],
    deps = [
        # Note: The target name inside the module might differ.
        # Usually it's //:main_lib or specific targets.
        "@googletest//:gtest_main",
        "@abseil-cpp//absl/strings",
    ],
)
```

---

## 4. Handling Non-Registry Dependencies

If a library isn't in the BCR, you can use `http_archive` via a module extension, but for most standard C++ needs (Boost, FMT, GTest, Protobuf), they are already in the BCR.

### Example: Searching for targets
To find out what targets are available in an external dependency (e.g., inside `abseil-cpp`), you can query:

```bash
bazel query @abseil-cpp//...
```
