# serde_json float_roundtrip arbitrary_precision

When using Rust crate serde_json, use serde_json crate features:

- "float_roundtrip" makes f64 -> JSON -> f64 produce output identical to the input.
- "arbitrary_precision" makes JSON -> serde_json::Number -> JSON produce output identical to the input.

Example in file `Cargo.toml` dependencies:

```toml
serde_json = { version = "…", features = ["float_roundtrip", "arbitrary_precision"] }
```
