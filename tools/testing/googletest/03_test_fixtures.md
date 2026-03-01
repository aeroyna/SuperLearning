# Test Fixtures

Test fixtures allow you to share data configuration between tests. If two or more tests operate on similar data, write a **Test Fixture**.

## Creating a Fixture

1.  Derive a class from `::testing::Test`.
2.  Declare any internal state (variables) you want to share.
3.  Implement `SetUp()` to verify/initialize state before each test.
4.  Implement `TearDown()` to clean up after each test.

```cpp
#include <gtest/gtest.h>
#include "queue.h"

class QueueTest : public ::testing::Test {
protected:
    void SetUp() override {
        // Runs before EACH test
        q1_.Enqueue(1);
        q2_.Enqueue(2);
        q2_.Enqueue(3);
    }

    void TearDown() override {
        // Runs after EACH test
        // Optional: Only needed if standard destructor isn't enough
    }

    Queue<int> q0_;
    Queue<int> q1_;
    Queue<int> q2_;
};
```

## Using a Fixture

Use `TEST_F()` (F for Fixture) instead of `TEST()`. The first argument must match the class name.

```cpp
// Access class members directly
TEST_F(QueueTest, IsEmptyInitially) {
    EXPECT_EQ(q0_.Size(), 0);
}

TEST_F(QueueTest, DequeueWorks) {
    int n = q1_.Dequeue();
    EXPECT_EQ(n, 1);
}
```

## Internal Lifecycle
1.  GTest constructs a **new** `QueueTest` object.
2.  Calls `SetUp()`.
3.  Runs the test body.
4.  Calls `TearDown()`.
5.  Destructs the object.

*Crucial: Internal state is NOT shared between tests. Each test gets a fresh object.*
