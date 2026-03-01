# GoogleTest (GTest) Complete Guide

A comprehensive guide to the industry-standard C++ testing framework.

## Topics

*   [**1. Setup and Integration**](01_setup_and_integration.md)
    *   CMake `FetchContent` integration (Best Practice).
    *   Running tests with `ctest`.
*   [**2. Assertions and Expectations**](02_assertions_and_expectations.md)
    *   `ASSERT` vs `EXPECT`.
    *   Binary and String comparisons.
    *   Exception testing.
*   [**3. Test Fixtures**](03_test_fixtures.md)
    *   Sharing setup code (`SetUp` / `TearDown`).
    *   `TEST_F` macro.
    *   Lifecycle of a fixture.
*   [**4. Parameterized Tests**](04_parameterized_tests.md)
    *   Data-driven testing with `TEST_P`.
    *   Generators: `Values`, `Range`, `Combine`.
*   [**5. Mocking with GoogleMock**](05_mocking_with_gmock.md)
    *   Mocking interfaces.
    *   `MOCK_METHOD` syntax.
    *   Setting expectations (`EXPECT_CALL`, `WillOnce`, `Return`).
*   [**6. Advanced Topics**](06_advanced_topics.md)
    *   Death Tests (`EXPECT_DEATH`).
    *   Type-Parameterized Tests.
    *   Testing private members (`FRIEND_TEST`).
