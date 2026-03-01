# Unit Testing Concepts

Unit testing involves testing individual components (functions, classes) of software in isolation.

## The AAA Pattern
A common structure for unit tests:

1.  **Arrange**: Set up the necessary objects and state.
2.  **Act**: Call the function or method being tested.
3.  **Assert**: Verify that the result matches the expectation.

## Example (Conceptual)

```cpp
void TestCalculatorAdd() {
    // Arrange
    Calculator calc;
    
    // Act
    int result = calc.add(2, 2);
    
    // Assert
    if (result != 4) {
        print("Failed!");
    }
}
```

## Why Unit Test in C++?
*   **Safety**: Catches compilation errors and logical bugs early.
*   **Refactoring**: Allows you to change implementation details (e.g., optimize loop) without breaking behavior.
*   **Documentation**: Tests show exactly how a class is intended to be used.
