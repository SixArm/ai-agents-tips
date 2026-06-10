# Rust .cargo/config.toml MUSL

Configure Rust compiler to build for Linux MUSL target.

Add to file .cargo/config.toml MUSL:

```toml
[target.x86_64-unknown-linux-musl]
linker = "musl-gcc"
```