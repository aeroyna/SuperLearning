# Hands-on: Building a "Greeter" Project 🏗️

This guide walks you through building a complete multi-package C++ project from scratch. We will build a simple app that uses a library to print a message, demonstrating visibility, headers, and dependency management.

---

## 📂 Project Structure

We will create the following file layout:

```text
/my-bazel-project
├── MODULE.bazel          <-- Workspace Root
├── lib/
│   ├── BUILD             <-- Defines the 'greeter' library
│   ├── greeter.h         <-- Public Interface
│   └── greeter.cc        <-- Implementation
└── main/
    ├── BUILD             <-- Defines the 'app' binary
    └── main.cc           <-- Application Entry Point
```

---

## 1. The Workspace Setup (`MODULE.bazel`)

This file marks the root directory. We will strictly use **Bzlmod**.

```python
# MODULE.bazel

# Define the project name and version
module(
    name = "my_greeter_app",
    version = "1.0",
)

# In a real project, we would add dependencies here:
# bazel_dep(name = "googletest", version = "1.14.0")
```

---

## 2. The Library Package (`lib/`)

We want to create a reusable library.

### `lib/greeter.h`
```cpp
#ifndef LIB_GREETER_H_
#define LIB_GREETER_H_

#include <string>

// A simple class to demonstrate structure
class Greeter {
public:
    explicit Greeter(std::string name);
    void SayHello() const;

private:
    std::string name_;
};

#endif // LIB_GREETER_H_
```

### `lib/greeter.cc`
```cpp
#include "lib/greeter.h" // Note the path! Root-relative is preferred.
#include <iostream>

Greeter::Greeter(std::string name) : name_(std::move(name)) {}

void Greeter::SayHello() const {
    // Implementation logic
    std::cout << "Hello, " << name_ << "! Welcome to Bazel." << std::endl;
}
```

### `lib/BUILD`
This is where the magic happens. We define a `cc_library` rule.

```python
# lib/BUILD

cc_library(
    name = "greeter_lib",
    
    # Public Headers: These are exposed to consumers.
    # Anyone depending on ":greeter_lib" can #include "lib/greeter.h"
    hdrs = ["greeter.h"],
    
    # Implementation: These are compiled but HIDDEN.
    # Consumers cannot see symbols or headers strictly inside here.
    srcs = ["greeter.cc"],
    
    # Visibility: Crucial!
    # By default, targets are PRIVATE.
    # We must explicitly allow //main to see this, or make it public.
    visibility = ["//main:__pkg__"], 
    # Or use ["//visibility:public"] to allow everyone.
)
```

---

## 3. The Application Package (`main/`)

Now we build the executable that consumes the library.

### `main/main.cc`
```cpp
#include "lib/greeter.h" // We can include this because it's in 'hdrs' of dependency

int main() {
    Greeter greeter("Developer");
    greeter.SayHello();
    return 0;
}
```

### `main/BUILD`

```python
# main/BUILD

cc_binary(
    name = "hello_world",
    
    # Source for this binary
    srcs = ["main.cc"],
    
    # Dependencies: We point to the target in the 'lib' package.
    # Syntax: //path/to/package:target_name
    deps = ["//lib:greeter_lib"],
)
```

---

## 4. Building and Running 🏃‍♂️

Open your terminal at the workspace root (`/my-bazel-project`).

### Build
```bash
bazel build //main:hello_world
```
*   Bazel analyzes the graph.
*   It sees `//main:hello_world` depends on `//lib:greeter_lib`.
*   It compiles `greeter.cc` -> `greeter.o` -> `libgreeter_lib.a`.
*   It compiles `main.cc` -> `main.o`.
*   It links `main.o` with `libgreeter_lib.a` to create the final binary.

### Run
```bash
bazel run //main:hello_world
```

**Output:**
```text
Hello, Developer! Welcome to Bazel.
```

---

## 5. Common Pitfalls to Avoid ⚠️

1.  **Wrong Include Paths**: Trying to `#include "greeter.h"` instead of `"lib/greeter.h"`. Bazel roots are workspace-relative by default.
2.  **Missing Visibility**: If `lib/BUILD` didn't have `visibility = ...`, the build would fail with "Target //lib:greeter_lib is not visible from //main:__pkg__".
3.  **Missing Dependency**: If `main/BUILD` didn't have `deps = ["//lib:greeter_lib"]`, the *compiler* might find the header (if files are co-located oddly), but the *linker* would fail with "Undefined reference to Greeter::SayHello".
