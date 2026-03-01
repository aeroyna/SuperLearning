# Labels & Targets 🎯

How do you refer to a specific file or rule in a massive project? Bazel uses **Labels**.

---

## 1. Label Anatomy

A label uniquely identifies a **Target**.

Syntax: `//path/to/package:target_name`

*   `//`: Starts at the Workspace root.
*   `path/to/package`: The directory containing the `BUILD` file.
*   `:`: Separator.
*   `target_name`: The `name` defined in the `BUILD` file.

### Examples

| Label | Meaning |
| :--- | :--- |
| `//src/utils:logger` | Target `logger` in `src/utils/BUILD` |
| `//src/utils` | Shortcut for `//src/utils:utils` (Target name matches folder) |
| `:my_target` | Relative label. Refers to a target in the *current* BUILD file. |
| `@my_dep//:lib` | Target `lib` in the external workspace `my_dep`. |

---

## 2. Types of Targets

### 1. File Targets 📄
Represents a source file.
*   Label: `//src/main.cc`
*   Bazel automatically creates these for every file in the package.

### 2. Rule Targets 🛠️
Represents a buildable artifact (binary, library, test).
*   Label: `//src:app_binary`
*   Defined explicitly in `BUILD` files.

---

## 3. Visibility 👁️

By default, targets are **Private** (only visible to other rules in the *same* `BUILD` file).

To allow other packages to depend on your code, you must set visibility:

```python
cc_library(
    name = "public_lib",
    srcs = ["lib.cc"],
    # Allow ANY package in the workspace to use this
    visibility = ["//visibility:public"],
)

cc_library(
    name = "internal_lib",
    srcs = ["internal.cc"],
    # Only allow specific packages
    visibility = ["//src/backend:__pkg__"],
)
```
