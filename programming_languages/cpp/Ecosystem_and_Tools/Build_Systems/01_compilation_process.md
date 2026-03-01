# The Compilation Process

Understanding how C++ source code becomes an executable is crucial for debugging linker errors and configuring build systems.

## The 4 Stages

### 1. Preprocessing (`g++ -E`)
*   Handles `#include` directives (copy-pastes file contents).
*   Expands Macros (`#define`).
*   Removes comments.
*   **Output**: Translation Unit.

### 2. Compilation (`g++ -S`)
*   Translates C++ code into **Assembly** code specific to the architecture (x86, ARM).
*   Performs optimizations.
*   **Output**: Assembly file (`.s`).

### 3. Assembly (`g++ -c`)
*   Translates Assembly into **Machine Code** (binary instructions).
*   **Output**: Object file (`.o` or `.obj`).
*   *Note*: The object file contains code but doesn't know *where* external functions are located.

### 4. Linking (`g++ -o`)
*   Combines multiple object files and libraries (`.a`, `.so`, `.lib`, `.dll`).
*   Resolves symbols (connects function calls to definitions).
*   **Output**: Executable binary.

## Common Errors
*   **Compiler Error**: Syntax errors, type mismatches (Stage 1-2).
*   **Linker Error**: `undefined reference to 'foo'` (Stage 4). Usually means you declared a function but didn't define it, or forgot to link the library containing it.
