# Rust Unit Testing

Rust has first-class support for testing built into the language and Cargo.

## The Test Module

Unit tests go in the *same file* as the code, usually in a submodule named `tests` annotated with `#[cfg(test)]`.

```rust
// lib.rs

pub fn add(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    // Import names from outer (for super) module
    use super::*;

    #[test]
    fn test_add() {
        assert_eq!(add(2, 2), 4);
    }

    #[test]
    #[should_panic]
    fn test_panic() {
        panic!("This test must panic to pass");
    }
}
```

## Running Tests

*   `cargo test`: Run all tests (unit, integration, doc).
*   `cargo test test_name`: Run specific test.
*   `cargo test -- --nocapture`: Show stdout even for passing tests.
