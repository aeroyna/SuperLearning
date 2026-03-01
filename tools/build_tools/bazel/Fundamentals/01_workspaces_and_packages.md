# Workspaces & Packages 📦

Bazel organizes code into a specific hierarchy. Understanding this is prerequisite #1.

---

## 1. The Workspace (Root)

A **Workspace** is a directory on your filesystem that contains the source files for the software you want to build. It is marked by a file named `MODULE.bazel` (modern) or `WORKSPACE` (legacy) at the root.

*   **Role**: Defines the project root and external dependencies.
*   **Best Practice**: Use `MODULE.bazel` (Bzlmod) for new projects starting Bazel 6.0+.

```text
/my-project
  ├── MODULE.bazel   <-- Marks this dir as a Workspace
  ├── src/
  └── lib/
```

---

## 2. Packages 📂

A **Package** is any subdirectory within the workspace that contains a file named `BUILD` (or `BUILD.bazel`).

*   A package includes all files in its directory and all subdirectories beneath it, **unless** those subdirectories contain their own `BUILD` file.
*   **Boundary**: A `BUILD` file creates a package boundary.

### Example Hierarchy
```text
/my-project
  ├── MODULE.bazel
  ├── BUILD           <-- Root Package
  ├── src/
  │   ├── BUILD       <-- Package 'src'
  │   └── main.cc
  └── utils/
      └── helper.cc   <-- Part of 'src' package? NO.
                          'utils' has no BUILD file, so helper.cc
                          belongs to the closest parent package.
                          If 'src/BUILD' exists, it belongs to 'src'.
```

---

## 3. The `BUILD` File

This file tells Bazel *how* to build the artifacts in that package. It uses a language called **Starlark** (a subset of Python).

```python
# src/BUILD

cc_binary(
    name = "my_app",
    srcs = ["main.cc"],
)
```
