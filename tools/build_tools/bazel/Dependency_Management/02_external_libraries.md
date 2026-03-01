# Integrating External Libraries 📚

Real-world C++ development often requires integration with common libraries. Here are patterns for the most popular ones.

---

## 1. GoogleTest (GTest)

**MODULE.bazel**
```python
bazel_dep(name = "googletest", version = "1.14.0")
```

**BUILD**
```python
cc_test(
    name = "unit_tests",
    srcs = ["test.cc"],
    deps = [
        "@googletest//:gtest",      # Core library
        "@googletest//:gtest_main", # Contains main() entry point
    ],
)
```

---

## 2. Abseil (Abseil-cpp)

Google's open-source collection of C++ library code.

**MODULE.bazel**
```python
bazel_dep(name = "abseil-cpp", version = "20230802.0")
```

**BUILD**
```python
cc_library(
    name = "my_lib",
    srcs = ["lib.cc"],
    deps = [
        "@abseil-cpp//absl/strings",
        "@abseil-cpp//absl/container:flat_hash_map",
    ],
)
```

---

## 3. Protocol Buffers (Protobuf)

**MODULE.bazel**
```python
bazel_dep(name = "protobuf", version = "24.4")
```

**BUILD**
```python
# To compile .proto files into C++
cc_proto_library(
    name = "my_proto_cc",
    deps = [":my_proto_lib"], # Depends on a proto_library rule
)
```

---

## 4. Boost

Boost is modularized in the BCR.

**MODULE.bazel**
```python
bazel_dep(name = "boost.filesystem", version = "1.83.0")
bazel_dep(name = "boost.asio", version = "1.83.0")
```

**BUILD**
```python
cc_binary(
    name = "app",
    deps = [
        "@boost.filesystem//:boost.filesystem",
        "@boost.asio//:boost.asio",
    ],
)
```
