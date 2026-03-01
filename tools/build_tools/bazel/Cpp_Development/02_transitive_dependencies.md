# Transitive Dependencies & Linking 🔗

Bazel handles dependency graphs explicitly.

---

## 1. How `deps` Work

When you add a target to `deps`:
1.  **Header Path**: The headers of the dependency are added to the include path.
2.  **Linking**: The compiled object code of the dependency is linked into your binary.

---

## 2. Transitive Dependencies

If `A` depends on `B`, and `B` depends on `C`:
*   **Linking**: `A` will link against `B` AND `C`. Bazel handles this automatically.
*   **Headers**:
    *   `A` can include headers from `B` (Direct dependency).
    *   `A` **cannot** include headers from `C` (Transitive dependency) unless `B` explicitly re-exports `C`.

### Strict Dependencies (Layering Check)
Bazel enforces **Strict Direct Dependencies**. You cannot include a header from a library you don't directly depend on in your `BUILD` file. This prevents "spooky action at a distance" where deleting an unused dependency breaks a distant target.

---

## 3. Static vs. Shared Libraries

By default, `cc_library` produces both static (`.a`) and dynamic (`.so`) outputs, but Bazel prefers **Static Linking** for `cc_binary` to create standalone executables.

### Forcing Shared Linking
If you want to force a dynamic library build:

```python
cc_binary(
    name = "dynamic_app",
    srcs = ["main.cc"],
    deps = [":my_lib"],
    linkstatic = False,  # Force dynamic linking
)
```

---

## 4. `alwayslink`

Sometimes a library has no symbols referenced by the main program (e.g., a library that registers plugins using global constructors). The linker might strip it out as "unused".

```python
cc_library(
    name = "my_plugin",
    srcs = ["plugin.cc"],
    alwayslink = True,  # Force linker to keep this
)
```
