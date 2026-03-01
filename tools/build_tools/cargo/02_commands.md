# Cargo Commands

Cargo manages the entire workflow.

## Basic Workflow
*   `cargo new my_project`: Create a new binary project.
*   `cargo new my_lib --lib`: Create a new library project.
*   `cargo build`: Compile debug build (`target/debug/`).
*   `cargo run`: Compile and run debug build.
*   `cargo build --release`: Compile optimized build (`target/release/`).
*   `cargo check`: Check code for errors *without* generating binaries (Standard "linting" step, much faster than build).

## Code Tools
*   `cargo fmt`: Format code using `rustfmt`.
*   `cargo clippy`: Run the Clippy linter (catches non-idiomatic code).
*   `cargo doc --open`: Generate and inspect documentation locally.

## Dependency Management
*   `cargo update`: Update `Cargo.lock` to latest compatible versions.
*   `cargo tree`: Visualize dependency graph.
