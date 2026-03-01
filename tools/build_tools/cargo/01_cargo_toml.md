# The Cargo Manifest (Cargo.toml)

`Cargo.toml` is the manifest file for Rust packages. It uses the TOML (Tom's Obvious, Minimal Language) format.

## Package Section

```toml
[package]
name = "my_project"
version = "0.1.0"
edition = "2021"
authors = ["Your Name <you@example.com>"]
description = "A short description"
license = "MIT"
```

## Dependencies

Dependencies can be sourced from Crates.io, git repositories, or local paths.

```toml
[dependencies]
# From Crates.io
serde = { version = "1.0", features = ["derive"] }
tokio = { version = "1.0", features = ["full"] }

# From Git
rand = { git = "https://github.com/rust-random/rand.git", rev = "0.8" }

# From Local Path
my-lib = { path = "../my-lib" }

[dev-dependencies]
# Only for tests/benchmarks
criterion = "0.4"

[build-dependencies]
# Only for build scripts (build.rs)
cc = "1.0"
```

## Profiles
Control compiler settings for different modes.

```toml
# "cargo build"
[profile.dev]
opt-level = 0
debug = true

# "cargo build --release"
[profile.release]
opt-level = 3
debug = false
```
