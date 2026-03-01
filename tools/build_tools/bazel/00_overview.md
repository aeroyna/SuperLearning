# Bazel for C++ Developers 🌿

**Bazel** is a multi-language, fast, and scalable build system developed by Google. It is the open-source version of their internal build system, **Blaze**.

It is famous for:
1.  **Speed**: Aggressive caching (local and remote) and incremental builds.
2.  **Correctness**: Hermetic builds ensure that if it builds on my machine, it builds on yours.
3.  **Scalability**: Handles codebases with millions of lines of code efficiently.

---

## 🗺️ Learning Path

This module is tailored specifically for **C++ engineers**. We skip the Java/Android fluff and focus on `cc_library`, `cc_binary`, and linking mechanics.

### 1. Fundamentals
Understand the core philosophy of Bazel: Workspaces, Packages, and the Graph.
*   [**1.1 Workspaces & Packages**](Fundamentals/01_workspaces_and_packages.md)
*   [**1.2 BUILD Files & Starlark**](Fundamentals/02_build_files_and_starlark.md)
*   [**1.3 Labels & Targets**](Fundamentals/03_labels_and_targets.md)

### 2. C++ Development Rules
Deep dive into the native C++ rules and best practices for structuring your code.
*   [**2.1 cc_binary & cc_library**](Cpp_Development/01_cc_binary_and_library.md)
*   [**2.2 Transitive Dependencies & Linking**](Cpp_Development/02_transitive_dependencies.md)
*   [**2.3 Includes & Visibility**](Cpp_Development/03_includes_and_visibility.md)

### 3. Dependency Management
How to bring in the outside world (GTest, Boost, Abseil) using the modern **Bzlmod** system.
*   [**3.1 Bzlmod & The Registry**](Dependency_Management/01_bzlmod_registry.md)
*   [**3.2 Integrating External Libraries**](Dependency_Management/02_external_libraries.md)

### 4. Hands-on Project 🛠️
Apply what you've learned by building a multi-package C++ application from scratch.
*   [**4.1 "Greeter" Project Walkthrough**](Hands_On_Example/01_full_walkthrough.md)

---

## 🚀 Quick Start Cheat Sheet

```bash
# Build a target
bazel build //src:my_app

# Run a binary
bazel run //src:my_app

# Test everything
bazel test //...

# Clean build artifacts
bazel clean

# Query the dependency graph (Who depends on my_lib?)
bazel query "rdeps(//..., //src:my_lib)"
```
