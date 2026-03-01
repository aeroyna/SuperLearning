# Targets & Properties 🎯

**Modern CMake** (3.0+) is all about **Targets**. Do not use global variables like `include_directories()` or `link_libraries()`.

---

## 1. What is a Target?

A target is a job for the build system. It corresponds to an executable or a library.

```cmake
# Create an executable target
add_executable(my_app main.cpp)

# Create a library target
add_library(my_lib src/lib.cpp)
```

---

## 2. Target Properties

Instead of global flags, you attach information *directly* to the target.

### Include Directories
Where are the headers for this target?
```cmake
target_include_directories(my_lib PUBLIC include)
```

### Compile Features
Does this target need C++17?
```cmake
target_compile_features(my_lib PUBLIC cxx_std_17)
```

### Compile Definitions
Does this target need a preprocessor macro?
```cmake
target_compile_definitions(my_lib PRIVATE DEBUG_MODE=1)
```

---

## 3. Linking Targets

This is the killer feature. When you link Target A to Target B, A automatically inherits B's usage requirements (headers, flags).

```cmake
# Link my_app to my_lib
target_link_libraries(my_app PRIVATE my_lib)
```
*   `my_app` will link against `libmy_lib.a`.
*   `my_app` will automatically add `my_lib`'s **PUBLIC** include directories to its own search path.
