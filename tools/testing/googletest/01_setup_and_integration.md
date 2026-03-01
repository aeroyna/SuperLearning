# Setup and Integration

GoogleTest (GTest) is a C++ testing framework developed by Google. It is designed to be portable, rigorous, and easy to use.

## Integrating with CMake

The modern and recommended way to include GTest is using CMake's `FetchContent` module, which downloads and builds GTest as part of your project.

### Step 1: `CMakeLists.txt`

```cmake
cmake_minimum_required(VERSION 3.14)
project(my_project)

# Set C++ Standard
set(CMAKE_CXX_STANDARD 14)

include(FetchContent)
FetchContent_Declare(
  googletest
  URL https://github.com/google/googletest/archive/refs/tags/v1.14.0.zip
)
# For Windows: Prevent overriding the parent project's compiler/linker settings
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)

FetchContent_MakeAvailable(googletest)

enable_testing()

add_executable(
  hello_test
  hello_test.cc
)
target_link_libraries(
  hello_test
  GTest::gtest_main
)

include(GoogleTest)
gtest_discover_tests(hello_test)
```

### Step 2: Running Tests

You can run the executable directly or use CTest:

```bash
mkdir build && cd build
cmake ..
cmake --build .
ctest --output-on-failure
```

### Pre-built Binaries (Not Recommended)
While you can install GTest system-wide (e.g., `apt-get install libgtest-dev`), this is discouraged because C++ has no standard ABI. Compiling GTest with different flags (e.g., C++11 vs C++17) than your project can cause link errors or crashes. **Always build GTest from source with your project.**
