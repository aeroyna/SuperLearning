# Documentation Tests

Rust automatically tests code examples inside documentation comments (`///`). This ensures your documentation never goes out of date.

## Syntax

```rust
/// Adds two numbers.
///
/// # Examples
///
/// ```
/// let result = my_crate::add(2, 2);
/// assert_eq!(result, 4);
/// ```
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

## Running
`cargo test` automatically runs these examples. If the example fails to compile or assertion fails, the test suite fails.
