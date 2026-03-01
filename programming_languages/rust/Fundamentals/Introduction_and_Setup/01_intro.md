# Introduction to Rust

Rust is a systems programming language that runs blazingly fast, prevents segfaults, and guarantees thread safety.

## Key Features
*   **Memory Safety without Garbage Collection**: Achieved via Ownership and Borrowing.
*   **Zero-Cost Abstractions**: High-level concepts compile down to efficient machine code.
*   **Concurrency without Data Races**: The compiler prevents race conditions.

## Installation
Use `rustup`, the official installer.

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

## Hello World

```rust
fn main() {
    println!("Hello, world!");
}
```
Compile with `rustc main.rs` or run with `cargo run`.
