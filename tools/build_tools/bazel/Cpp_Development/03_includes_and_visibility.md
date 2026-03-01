# Includes & Visibility 🔍

Managing include paths and access control is vital for large C++ codebases.

---

## 1. Include Paths

By default, Bazel expects include paths to be relative to the **Workspace Root**.

### Example Structure
```text
/workspace
  /libs
    /math
      geometry.h
```

**Correct Include:**
```cpp
#include "libs/math/geometry.h"
```

**Incorrect Include:**
```cpp
#include "geometry.h" // Error: File not found
```

### The `includes` Attribute (Strip Include Prefix)
If you want to support shorter include paths (legacy code or library conventions), use the `includes` attribute.

```python
# libs/math/BUILD
cc_library(
    name = "math",
    hdrs = ["geometry.h"],
    # Adds 'libs/math' to the compiler's -I flags (system include search path)
    includes = ["."], 
)
```

Now consumers can do:
```cpp
#include "geometry.h"
```

> **Warning**: Use `includes` sparingly. Root-relative paths are the "Bazel Way" as they prevent naming collisions (e.g., two libraries both having `utils.h`).

---

## 2. `strip_include_prefix`

A cleaner alternative to `includes`. It creates a virtual include root.

```python
cc_library(
    name = "math",
    hdrs = ["src/include/math/geometry.h"],
    strip_include_prefix = "src/include",
)
```
The file effectively becomes `math/geometry.h` for consumers.

---

## 3. Visibility (Advanced)

We covered `public` vs `private`. There are nuance options:

### Package Group
Define a group of packages that are "friends".

```python
# //BUILD (Root)
package_group(
    name = "backend_team",
    packages = [
        "//server/...",
        "//database/...",
    ],
)
```

```python
# //server/auth/BUILD
cc_library(
    name = "secret_auth",
    srcs = ["auth.cc"],
    visibility = ["//:backend_team"],
)
```
