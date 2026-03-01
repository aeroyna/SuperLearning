# Parameterized Tests

Parameterized tests allow you to run the same test logic over different input values without duplicating code.

## 1. Create a Test Class
Must inherit from `::testing::TestWithParam<T>`, where `T` is the parameter type.

```cpp
class MathTest : public ::testing::TestWithParam<int> {
    // You can implement SetUp/TearDown here too
};
```

## 2. Write the Logic
Use `TEST_P()` (P for Parameterized). Use `GetParam()` to access the current value.

```cpp
TEST_P(MathTest, IsEven) {
    int n = GetParam();
    EXPECT_TRUE(n % 2 == 0);
}
```

## 3. Instantiate the Suite
Tell GTest which values to use.

```cpp
INSTANTIATE_TEST_SUITE_P(
    EvenNumbers,        // Instance Name (arbitrary)
    MathTest,           // Test Class Name
    ::testing::Values(2, 4, 6, 8, 100)
);
```

## Advanced Parameter Generators

*   `Values(v1, v2, ...)`: Explicit list.
*   `Range(start, end [, step])`: Integers from start up to (excluding) end.
*   `Bool()`: Returns `true` and `false`.
*   `Combine(g1, g2, ...)`: Cartesian product of generators (requires `std::tuple` param).
