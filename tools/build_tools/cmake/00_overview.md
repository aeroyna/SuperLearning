# Modern CMake Guide 🛠️

**CMake** is the de-facto standard build system for C++. It is not a build system itself (like Make or Ninja), but a **generator**. It generates the build files for your native toolchain.

---

## 🗺️ Learning Path

This module focuses on **Modern CMake** (versions 3.15+). We avoid legacy practices (like global variables) and focus on the "Target-Based" philosophy.

### 1. Fundamentals
The syntax, variables, and flow control.
*   [**1.1 Syntax & Variables**](Fundamentals/01_syntax_and_variables.md)

### 2. Modern CMake (Target-Based)
The core philosophy: Treat everything as a "Target" with properties.
*   [**2.1 Targets & Properties**](Modern_CMake/01_targets_and_properties.md)
*   [**2.2 Public, Private, & Interface**](Modern_CMake/02_public_private_interface.md)

### 3. Dependency Management
How to find system libraries or download dependencies on the fly.
*   [**3.1 FetchContent & find_package**](Dependency_Management/01_fetchcontent_and_findpackage.md)

### 4. Hands-on Project 🏗️
Build a complete C++ project with library and application separation.
*   [**4.1 "Greeter" Project Walkthrough**](Hands_On_Example/01_full_walkthrough.md)

---

## 🚀 Quick Start Cheat Sheet

```bash
# 1. Configure
cmake -S . -B build

# 2. Build
cmake --build build

# 3. Test (if CTest is enabled)
cd build && ctest
```