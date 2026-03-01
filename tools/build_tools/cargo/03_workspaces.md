# Workspaces

Cargo workspaces allow you to manage multiple packages (crates) in a single repository, sharing a `Cargo.lock` and output directory.

## Structure

```text
root/
  Cargo.toml (Workspace Root)
  Cargo.lock
  crates/
    api/
      Cargo.toml
    core/
      Cargo.toml
```

## Root Cargo.toml

```toml
[workspace]
members = [
    "crates/api",
    "crates/core",
]
# Common dependencies for all members (Rust 1.64+)
[workspace.dependencies]
serde = "1.0"
```

## Member Cargo.toml

```toml
[package]
name = "api"
version = "0.1.0"

[dependencies]
# Inherit from workspace
serde = { workspace = true }
# Local dependency
core = { path = "../core" }
```
