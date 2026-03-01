# Hands-on: Building a "Greeter" Project 🏗️

This guide walks you through building a complete multi-directory C++ project using Modern CMake.

---

## 📂 Project Structure

```text
/my-cmake-project
├── CMakeLists.txt          <-- Root CMake
├── include/
│   └── greeter.h           <-- Public Interface
├── src/
│   ├── CMakeLists.txt      <-- Library Build Definition
│   └── greeter.cpp         <-- Implementation
└── app/
    ├── CMakeLists.txt      <-- Application Build Definition
    └── main.cpp            <-- Entry Point
```

---

## 1. The Root `CMakeLists.txt`

```cmake
cmake_minimum_required(VERSION 3.15)

project(GreeterApp VERSION 1.0 LANGUAGES CXX)

# Standard: Use C++17
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Add subdirectories
# Order matters: Dependencies first!
add_subdirectory(src)
add_subdirectory(app)
```

---

## 2. The Library (`src/`)

### `src/greeter.cpp`
```cpp
#include "greeter.h"
#include <iostream>

void Greeter::sayHello() {
    std::cout << "Hello from CMake!" << std::endl;
}
```

### `include/greeter.h`
```cpp
#pragma once
class Greeter {
public:
    void sayHello();
};
```

### `src/CMakeLists.txt`
```cmake
# Create the library target
add_library(greeter_lib greeter.cpp)

# Define include directories
target_include_directories(greeter_lib 
    PUBLIC 
        "${PROJECT_SOURCE_DIR}/include" # Consumers need this
    PRIVATE 
        "${CMAKE_CURRENT_SOURCE_DIR}"   # Only I need this
)
```

---

## 3. The Application (`app/`)

### `app/main.cpp`
```cpp
#include "greeter.h"

int main() {
    Greeter g;
    g.sayHello();
    return 0;
}
```

### `app/CMakeLists.txt`
```cmake
add_executable(my_app main.cpp)

# Link against the library
# This AUTOMATICALLY adds "${PROJECT_SOURCE_DIR}/include" to include path!
target_link_libraries(my_app PRIVATE greeter_lib)
```

---

## 4. Build & Run 🏃‍♂️

Modern CMake uses an "out-of-source" build workflow.

```bash
# 1. Configure (Generate Build System)
# -S . : Source is here
# -B build : Build in 'build' folder
cmake -S . -B build

# 2. Build (Compile)
cmake --build build

# 3. Run
./build/app/my_app
```

**Output:**
```text
Hello from CMake!
```
