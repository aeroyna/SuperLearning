# Dependency Management 📦

How to bring in 3rd party libraries like GoogleTest, FMT, or JSON.

---

## 1. FetchContent (Modern & Recommended)

Available since CMake 3.11, `FetchContent` allows you to download dependencies at configure time. It's similar to `git submodule` but managed by CMake.

```cmake
include(FetchContent)

# 1. Declare the dependency
FetchContent_Declare(
  googletest
  GIT_REPOSITORY https://github.com/google/googletest.git
  GIT_TAG        v1.14.0
)

# 2. Make it available (downloads & adds targets)
FetchContent_MakeAvailable(googletest)

# 3. Use it
add_executable(my_test test.cpp)
target_link_libraries(my_test PRIVATE GTest::gtest_main)
```

---

## 2. find_package (System Libraries)

Used to find libraries already installed on the system (e.g., via `apt-get`, `brew`, or `vcpkg`).

```cmake
# Looks for specific cmake config files in system paths
find_package(OpenCV 4.0 REQUIRED)

add_executable(my_vision_app main.cpp)

# Link against the imported target
target_link_libraries(my_vision_app PRIVATE opencv_core opencv_highgui)
```

---

## 3. Package Managers (Conan / Vcpkg)

For serious C++ development, it is highly recommended to use a dedicated package manager.

*   **Vcpkg**: Microsoft's package manager. Integrates with CMake via a toolchain file.
    ```bash
    cmake -B build -S . -DCMAKE_TOOLCHAIN_FILE=/path/to/vcpkg/scripts/buildsystems/vcpkg.cmake
    ```
*   **Conan**: Python-based package manager. Generates `Find*.cmake` files for you.
