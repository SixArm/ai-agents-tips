### Technology stack for Rust Loco

| Component            | Technology                                        | Purpose                                     |
| -------------------- | ------------------------------------------------- | ------------------------------------------- |
| **Language**         | Rust 1.95+ 2024 Edition                           | Systems programming, performance, safety    |
| **Async Runtime**    | Tokio                                             | Asynchronous I/O and concurrency            |
| **HTTP**             | tower-http                                        | HTTP layer, CORS, compression               |
| **Web Framework**    | Axum, Loco                                        | HTTP server and routing (backend-only)      |
| **Database**         | PostgreSQL 18+                                    | Data persistence                            |
| **ORM**              | SeaORM                                            | Async database object-relational mapper     |
| **Search Engine**    | Tantivy                                           | Full-text search indexing                   |
| **Data Streaming**   | Fluvio                                            | Event publishing and durable data streaming |
| **API Docs**         | Utoipa                                            | OpenAPI 3.0 specification                   |
| **Swagger Docs **    | utoipa-swagger-ui                                 | OpenAPI 3.0 specification                   |
| **Serialization**    | Serde                                             | JSON serialization/deserialization          |
| **Logging**          | Tracing                                           | Structured logging                          |
| **Observability**    | OpenTelemetry, opentelemetry-semantic-conventions | Distributed tracing, metrics, spans, logs   |
| **String Matching**  | strsim, fuzzy-matcher                             | Jaro-Winkler, Levenshtein                   |
| **Geo**              | geo, haversine                                    | Coordinate distance calculations            |
| **Containerization** | Podman                                            | Deployment packaging                        |
| **gRPC**             | Tonic                                             | High-performance RPC framework              |
| **Protocols**        | Prost                                             | Protocol buffers                            |
| **Timestamps**       | jiff                                              | Dates, times, durations                     |
| **Environment**      | dotenvy                                           | Config env var                              |
| **Error Handling**   | thiserror, anyhow                                 | Typed and contextual error handling         |
| **Security**         | argon2                                            | Password hashing                            |
| **Authentication**   | jsonwebtoken                                      | JWT authentication                          |
| **Testing**          | assertables, tokio-test, mockall, tempfile        | Unit, integration, and mock testing         |
| **Benchmarking**     | Criterion                                         | Statistical performance benchmarking        |
| **Memory Allocator** | MiMalloc                                          | High-performance MUSL static builds         |
| **Numbers**          | bigdecimal                                        |                                             |
| **Identifiers**      | uuid                                              | UUID generation                             |
| **Validation**       | validator                                         | Declarative validation                      |

[Configure main.rs for MUSL MiMalloc:

```rust
#[cfg(target_env = "musl")]
#[global_allocator]
static GLOBAL: mimalloc::MiMalloc = mimalloc::MiMalloc;
```

](Constraints:

- Podman NOT Docker (because Podman is more open source and more free)
- Tokio NOT async_std (because we harmonize Loco, Tower, Axum)
- MiMalloc NOT jemalloc (because MiMalloc handles multiple page sizes)
- PostgreSQL NOT SQLite (because we want testing on the same database)
- jiff NOT chrono (because jiff is better-designed and more-powerful)
- sea-orm feature "with-jiff" NOT "with-chrono" (jiff support is coming)
- rustls NOT OpenSSL (because rustls is easier for static compile)

## Configurations

Add to files `lib.rs` `main.rs` immediately after top-level doc comment.

```rust
// Always start with high quality coding conventions.
#![forbid(unsafe_code)]
#![deny(missing_docs)]
#![warn(clippy::clippy::pedantic)]

// When we build for MUSL static, use faster memory allocator.
#[cfg(target_env = "musl")]
#[global_allocator]
static GLOBAL: mimalloc::MiMalloc = mimalloc::MiMalloc;
```
