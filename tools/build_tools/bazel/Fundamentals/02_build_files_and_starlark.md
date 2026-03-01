# BUILD Files & Starlark 🐍

Bazel uses **Starlark**, a dialect of Python, for its configuration files. It is designed to be deterministic and parallelizable.

---

## 1. Syntax Basics

If you know Python, you know Starlark.
*   **Variables**: `x = "hello"`
*   **Lists**: `srcs = ["a.cc", "b.cc"]`
*   **Functions**: `glob(["*.cc"])`

**Key Difference**: You cannot modify variables after they are defined (immutability). This ensures thread safety during the loading phase.

---

## 2. Rules 📏

A `BUILD` file consists of **Rules**. A rule specifies the relationship between input files and output files.

### Common Structure
```python
<rule_kind>(
    name = "<target_name>",
    srcs = ["<source_file_1>", ...],
    deps = ["<dependency_target_1>", ...],
    visibility = ["//visibility:public"],
)
```

*   **name**: A unique identifier for the target within the package.
*   **srcs**: The input files (C++ source, headers).
*   **deps**: Other targets (libraries) this rule needs to link against.
*   **visibility**: Who can depend on this target?

---

## 3. Globbing 🌐

Instead of listing every file manually, use `glob()`.

```python
cc_library(
    name = "my_lib",
    srcs = glob(["*.cc"]),
    hdrs = glob(["*.h"]),
)
```
*   **Best Practice**: Avoid recursive globs (`**/*.cc`) if possible, as it blurs package boundaries.

---

## 4. Macros vs. Rules

*   **Rule**: Defines a build action (e.g., `cc_binary`). Built into Bazel or loaded from `.bzl` files.
*   **Macro**: A Starlark function that instantiates rules. Useful for reducing boilerplate.

```python
# my_macro.bzl
def my_custom_binary(name):
    native.cc_binary(
        name = name,
        srcs = ["main.cc"],
        copts = ["-Werror"],
    )
```
