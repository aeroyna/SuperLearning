# Advanced Topics

## Death Tests
Verify that the program crashes (e.g., via `assert` or `exit()`) when appropriate.

```cpp
TEST(MyDeathTest, ExitsOnBadInput) {
    // Verifies the process dies with an error message matching the regex
    EXPECT_DEATH(MyCrucialFunc(nullptr), "Error: Input cannot be null");
}
```
*Note: Death tests run in a forked process. Thread safety can be tricky.*

## Values Parameterized Tests
Used for Type-Parameterized tests (same test logic, different classes).

```cpp
template <typename T>
class StackTest : public ::testing::Test { ... };

using MyTypes = ::testing::Types<int, float, double>;
TYPED_TEST_SUITE(StackTest, MyTypes);

TYPED_TEST(StackTest, IsEmptyInitially) {
    TypeParam n = 0;
    // ...
}
```

## Private Member Testing
Generally discouraged (test public APIs!). However, if needed:
1.  **Friend Fixture**: Make the fixture class a `friend` in your production code.
2.  **FRIEND_TEST macro**:
    ```cpp
    class MyClass {
    private:
        int secret_calc(int x);
        FRIEND_TEST(MyClassTest, TestSecretCalc);
    };
    ```

## Event Listeners
You can hook into test lifecycle events (start of test, end of test, failure) to generate custom reports or clean up global state.
