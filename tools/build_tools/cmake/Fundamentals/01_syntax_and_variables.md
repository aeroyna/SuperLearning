# CMake Syntax & Variables 🔡

CMake is a build system generator. It reads `CMakeLists.txt` files and generates a build system (Makefiles, Ninja, Visual Studio Project) for your platform.

---

## 1. The Basics

Every CMake project starts with a `CMakeLists.txt` file at the root.

```cmake
# 1. Require a minimum version (Crucial for Modern CMake)
cmake_minimum_required(VERSION 3.15)

# 2. Define the project
project(
    MyProject
    VERSION 1.0
    DESCRIPTION "A simple C++ project"
    LANGUAGES CXX
)
```

---

## 2. Variables

Variables in CMake are strings.

### Setting Variables
```cmake
set(MY_VAR "Hello World")
set(SOURCES main.cpp utils.cpp) # A list (semicolon-separated string)
```

### Accessing Variables
Use `${}` to dereference.
```cmake
message(STATUS "Value: ${MY_VAR}")
```

### Cache Variables
These are variables that can be set from the command line (e.g., `-DENABLE_TESTS=ON`).
```cmake
set(ENABLE_TESTS ON CACHE BOOL "Enable unit tests")
```

---

## 3. Flow Control

### If / Else
```cmake
if(ENABLE_TESTS)
    message(STATUS "Tests are enabled")
else()
    message(STATUS "Tests are disabled")
endif()
```

### Loops
```cmake
set(MY_LIST a b c)
foreach(ITEM ${MY_LIST})
    message(STATUS "Item: ${ITEM}")
endforeach()
```
