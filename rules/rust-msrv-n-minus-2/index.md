# Rust MSRV — current N-2

This workspace's **Minimum Supported Rust Version (MSRV)** is **the current
stable Rust release minus two**: if the current stable release is `1.N`, the
MSRV is `1.(N-2)`.

This is a project policy that governs the Rust
toolchain the code in this workspace may assume.

## The rule

- Let `1.N.0` be the latest stable Rust release published by the Rust project.
- The MSRV MUST be `1.(N-2).0`.
- Code, tests, benchmarks, fuzz targets, and examples MUST compile with the
  MSRV toolchain. A language or standard-library feature stabilized after the
  MSRV MUST NOT be used.
- Only the minor version is pinned. Patch releases of the MSRV minor version
  (`1.(N-2).x`) are all acceptable; the recorded value uses `.0`.
- Pre-release channels (beta, nightly) are never the MSRV and MUST NOT be
  required by any workspace target, including the fuzz targets — see
  [rust-fuzz.md](../rust-fuzz.md), which keeps the nightly-only fuzz crate outside
  the workspace precisely so this rule holds.

## Where the MSRV is recorded

| Location                             | Form                                                   |
| ------------------------------------ | ------------------------------------------------------ |
| `Cargo.toml` (`[workspace.package]`) | `rust-version = "1.(N-2)"`                             |
| each `crates/*/Cargo.toml`           | `rust-version.workspace = true`                        |
| `.github/workflows/ci.yml`           | an `msrv` job pinning `dtolnay/rust-toolchain@1.(N-2)` |

`rust-version` is the single source of truth inside the workspace: `cargo`
refuses to build a crate with a toolchain older than it, and downstream
consumers see it in the published crate metadata. Every member crate inherits
it from `[workspace.package]`; a member MUST NOT declare its own value.

## Maintenance

When a new stable Rust release `1.N` appears, the MSRV becomes `1.(N-2)`
**in the same change** that observes the release:

1. Set `rust-version` in the root `Cargo.toml` to `1.(N-2)`.
2. Set the pinned toolchain in the CI `msrv` job to the same value.
3. Run `cargo +1.(N-2) check --all-targets --workspace` and fix anything that
   the older toolchain rejects — the MSRV is a floor the code must meet, not a
   ceiling on what the code may need.

Raising the MSRV is therefore routine and expected, not a breaking change to
be avoided. Lowering it below N-2 (to support an older consumer) is a design
decision for `plan.md`, not a convenience.

## CI enforcement

CI MUST verify the MSRV, not merely declare it. The `msrv` job installs the
exact pinned toolchain and runs `cargo check --all-targets --workspace` with
it. `cargo check` (not `cargo build`) is sufficient and fast: the MSRV question
is "does this compile", and the `test` job already answers "does this work" on
stable.

The `msrv` job is separate from the `test` job so a failure names the cause
directly: `test` red means a behavior regression, `msrv` red means the code
started requiring a newer toolchain than the policy allows.

## Current value

As of the most recent update to this document, stable Rust is **1.98**, so the
MSRV is **1.96**. If stable has moved on since, this document is stale in its
example only — the rule above is what binds, and `Cargo.toml` must be brought
back in line with it.
