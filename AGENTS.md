# AGENTS.md - SoulBridge Development Guide

## Project Overview

SoulBridge is a Rust workspace with three crates:
- **soul_msgs**: Message types for pub/sub communication
- **soul_macros**: Procedural macros  
- **soul_node**: Main node library using Zenoh and tokio

---

## Build, Lint, and Test Commands

### Build Commands
```bash
cargo build                    # Build entire workspace
cargo build -p soul_node       # Build specific crate
cargo build --release          # Release build
```

### Test Commands
```bash
cargo test                     # Run all tests
cargo test -p soul_node        # Run tests in specific crate
cargo test test_name           # Run single test by name (partial match)
cargo test soul_node::base::test::declare_publisher  # Run specific test
cargo test --test local_echo   # Run integration test
cargo test -- --nocapture      # Run with output
# Note: Tests use #[tokio::test(flavor = "multi_thread", worker_threads = 1)]
```

### Linting and Formatting
```bash
cargo fmt                      # Format code
cargo fmt -- --check           # Check formatting
cargo clippy                  # Run Clippy lints
cargo clippy -- -D warnings   # Warnings as errors
cargo check                   # All checks (fmt + clippy + test)
```

---

## Code Style Guidelines

### General Conventions

- **Rust Edition**: 2024 (`edition = "2024"`)
- **Naming**: PascalCase for types, snake_case for functions/variables
- **Async**: Use `tokio`, mark tests with `#[tokio::test(flavor = "multi_thread", worker_threads = 1)]`

### Error Handling
```rust
#[derive(Debug, Error)]
pub enum SoulNodeError {
    #[error("Failed in Zenoh. {0}")]
    Zenoh(#[from] zenoh::Error),
}
```
- Use `thiserror` with `#[derive(Error)]`
- Use `#[error("...")]` for messages, `#[from]` for conversions
- Use `anyhow` for application errors
- Use `Box<dyn std::error::Error + Send + Sync>` for generic errors

### Imports Order
1. Standard library (`std`)
2. External crates (`tokio`, `serde`, `thiserror`, `zenoh`)
3. Local crate modules (`crate::`)

### Types and Generics
- Use `Arc` for shared ownership: `topic_registry: Arc<TopicRegistry>`
- Use `dyn` for trait objects
- Generic bounds on separate lines:
```rust
impl<T> Publisher<T>
where
    T: zenoh_ext::Serialize + Debug + 'static,
{
```

### Documentation
- Doc comments use `///` format
- Add to public APIs, especially builders

### Testing
- Unit tests in `mod test` blocks within source files
- Integration tests in `tests/` directory
- Use `common` module for shared utilities

### Builder Pattern
```rust
let publisher = node
    .declare_publisher(topic_id)
    .build()
    .await
    .unwrap();
```

### Zenoh Integration
- Session: `zenoh::open(config).await?`
- Key expressions: `KeyExpr` for topic paths
- Serialization: `zenoh_ext::Serialize`, `zenoh_ext::Deserialize`
- Use `z_serialize(data)` for serialization

---

## Project Structure

```
soul_bridge/
├── Cargo.toml              # Workspace config
├── crates/
│   ├── soul_msgs/          # Message types
│   ├── soul_macros/       # Procedural macros
│   └── soul_node/         # Main library
│       ├── src/
│       │   ├── lib.rs
│       │   ├── base.rs
│       │   ├── pub_sub.rs
│       │   └── topic.rs
│       └── tests/
│           ├── local_echo.rs
│           ├── liveliness.rs
│           └── common/mod.rs
```

---

## Common Development Tasks

### Adding a New Feature
1. Add dependencies to appropriate `Cargo.toml`
2. Create/modify source files in `src/`
3. Add unit tests in `mod test` blocks
4. Add integration tests in `tests/` if needed
5. Run `cargo fmt` and `cargo clippy`

### Running a Specific Test
```bash
cargo test -p soul_node --test local_echo
```

### Debugging
```bash
RUST_LOG=debug cargo test    # Enable logging
cargo test -- --nocapture    # Run with output
```
