# JUnit 5 Basics

JUnit 5 (Jupiter) is the next generation of JUnit.

## Key Annotations

*   **`@Test`**: Denotes a method as a test method.
*   **`@BeforeEach` / `@AfterEach`**: Runs before/after *each* test method.
*   **`@BeforeAll` / `@AfterAll`**: Runs once before/after *all* tests in the class. Must be static (unless using `@TestInstance(Lifecycle.PER_CLASS)`).
*   **`@DisplayName("Custom Name")`**: Sets a readable name for reports.
*   **`@Disabled`**: Skips the test.

## Example

```java
import org.junit.jupiter.api.*;

class CalculatorTest {

    static Calculator calc;

    @BeforeAll
    static void init() {
        calc = new Calculator();
    }

    @Test
    @DisplayName("1 + 1 = 2")
    void addsTwoNumbers() {
        Assertions.assertEquals(2, calc.add(1, 1));
    }
}
```
