# CMake

> [!NOTE]
> For a comprehensive deep-dive into CMake, including Modern CMake patterns and advanced usage, see the dedicated [**CMake Tools Section**](../../../../tools/build_tools/cmake/00_overview.md).


CMake is a meta-build system. It doesn't build code itself; it **generates** build files for other systems (Makefiles, Ninja, Visual Studio). It is the industry standard for C++.

## Modern CMake (Target-Based)
Avoid global variables. Think in terms of **Targets** (executables, libraries).

### `CMakeLists.txt`

```cmake
cmake_minimum_required(VERSION 3.10)
project(MyProject)

# Set C++ Standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# 1. Create a Library
add_library(MathLib
    src/math.cpp
    include/math.h
)
# Define public headers (so consumers can include them)
target_include_directories(MathLib PUBLIC include)

# 2. Create an Executable
add_executable(MyApp main.cpp)

# 3. Link them
target_link_libraries(MyApp PRIVATE MathLib)
```

## How to Build

```bash
mkdir build
cd build
cmake ..        # Generate build files
cmake --build . # Compile
```

## Key Commands
*   `add_executable`: Defines a binary top-level target.
*   `add_library`: Defines a static (`.a/.lib`) or shared (`.so/.dll`) library.
*   `target_include_directories`: Where `#include` looks for headers.
*   `target_link_libraries`: Dependency management.
