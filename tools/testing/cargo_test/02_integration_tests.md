# Integration Tests

Integration tests are external to your library. They use your library exactly like any other code would (via `use`).

## The `tests/` Directory

Cargo treats any file in the `tests/` directory as a separate crate.

```text
my_project/
  src/
    lib.rs
  tests/
    integration_test.rs
```

## Example `integration_test.rs`

```rust
use my_project;

#[test]
fn test_public_api() {
    assert_eq!(my_project::add(3, 4), 7);
}
```

## Shared Setup
To share code between integration tests (helper functions), create `tests/common/mod.rs` so Cargo doesn't treat it as a test file.
