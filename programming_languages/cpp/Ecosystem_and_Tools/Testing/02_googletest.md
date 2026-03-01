# GoogleTest (GTest)

> [!NOTE]
> For a comprehensive deep-dive into GoogleTest, including mocking (gMock), parameterized tests, and advanced CMake integration, see the dedicated [**GoogleTest Tools Section**](../../../../tools/testing/googletest/00_overview.md).


GoogleTest is the most popular C++ testing framework.

## Setup
GTest is usually added via CMake (using `FetchContent` or `find_package`).

## Basic Test Structure

```cpp
#include <gtest/gtest.h>

// Function to test
int add(int a, int b) { return a + b; }

// define a test case
TEST(TestLinks, Addition) {
    EXPECT_EQ(add(1, 1), 2);
    EXPECT_EQ(add(-1, 1), 0);
}

int main(int argc, char **argv) {
    ::testing::InitGoogleTest(&argc, argv);
    return RUN_ALL_TESTS();
}
```

## Assertions
*   `ASSERT_EQ(val1, val2)`: Fatal failure. Aborts the current test function.
*   `EXPECT_EQ(val1, val2)`: Non-fatal. Reports failure but continues (useful to see multiple errors).

Common variants: `_NE` (Not Equal), `_TRUE`, `_FALSE`, `_GT` (Greater Than).

## Test Fixtures
Used when multiple tests need the same setup.

```cpp
class DatabaseTest : public ::testing::Test {
protected:
    void SetUp() override {
        // Connect to DB
        db.connect();
    }

    void TearDown() override {
        // Disconnect
        db.disconnect();
    }
    
    Database db;
};

TEST_F(DatabaseTest, InsertQueries) {
    // Has access to 'db'
    db.insert("user", "Alice");
    EXPECT_EQ(db.count("user"), 1);
}
```
*Note use of `TEST_F` (F for Fixture) instead of `TEST`.*
