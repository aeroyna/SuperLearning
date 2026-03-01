# Parameterized Tests

Run the same test with different arguments. Requires `junit-jupiter-params` artifact.

## @ValueSource
Simple literal values.

```java
@ParameterizedTest
@ValueSource(ints = { 1, 3, 5, -3, 15 })
void isOdd_ShouldReturnTrueForOddNumbers(int number) {
    assertTrue(Numbers.isOdd(number));
}
```

## @MethodSource
Complex objects from a factory method.

```java
@ParameterizedTest
@MethodSource("stringProvider")
void testWithExplicitLocalMethodSource(String argument) {
    assertNotNull(argument);
}

static Stream<String> stringProvider() {
    return Stream.of("apple", "banana");
}
```

## @CsvSource
Multiple arguments.

```java
@ParameterizedTest
@CsvSource({
    "apple,         1",
    "banana,        2",
    "'lemon, lime', 0xF1"
})
void testWithCsvSource(String fruit, int rank) {
    assertNotNull(fruit);
    assertNotEquals(0, rank);
}
```
