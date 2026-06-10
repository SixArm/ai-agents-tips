# AGENTS instructions

## Spec-driven development — single source of truth

Every service crate carries its own `spec.md` as the **canonical
artefact**. Code conforms to the spec; not the other way around.

- A behavioural change PR has **three parts**: spec edit + code edit
  + test edit. All in one PR.
- The live work queue for each crate lives in its `spec.md §13`.
- Open questions live in `spec.md §16` until resolved.
- See `<crate>/AGENTS/spec-driven-development.md` for the discipline.

Per-crate `spec.md`:

- [person-service/spec.md](../../person-service-rust-crate/spec.md)
- [event-service/spec.md](../../event-service-rust-crate/spec.md)
- [worker-service/spec.md](../../worker-service-rust-crate/spec.md)
- [place-service/spec.md](../../place-service-rust-crate/spec.md)
- [thing-service/spec.md](../../thing-service-rust-crate/spec.md)

## Overview

- Professional: Enterprise-grade, high-performance, production-ready
- RESTful API: Modern HTTP API with OpenAPI/Swagger documentation
- gRPC API: High-performance Remote Procedure Call (RPC) framework
- Data Quality: Validation, standardization, normalization, merging
- Matching: Probabilistic and deterministic matching algorithms
- Full-Text Search: Powered by Tantivy for fast, accurate record searches
- Duplicate Detection: Real-time and batch deduplication with review queue
- Record Merging: Merge confirmed duplicate records with full audit trail
- Create, read, update, delete (CRUD) records
- Soft delete support with complete audit trails
- Privacy: Data masking, GDPR data export, consent management
- Event Streaming: Real-time event publishing with audit logging
- Database Integration: PostgreSQL with SeaORM and migrations
- Docker Ready: Multi-stage builds, Docker Compose for dev/test/prod
- Testing: Comprehensive unit tests, integration tests, benchmark tests
- Observability with tracing, log levels, and OpenTelemetry
- Production Hardened: Security, monitoring, and compliance features

## agents/share

| File                                                             | Description                                            |
| ---------------------------------------------------------------- | ------------------------------------------------------ |
| [overview.md](overview.md)                                       | High-level project overview (consumed by `@` includes) |
| [architecture.md](architecture.md)                               | Architecture                                           |
| [auditability.md](auditability.md)                               | Audit logging and event streaming                      |
| [availability.md](availability.md)                               | Availability, scaling, health checks                   |
| [dataflow.md](dataflow.md)                                       | Create / match / merge / search data flows             |
| [match.md](match.md)                                             | Matching algorithms                                    |
| [search.md](search.md)                                           | Search (Tantivy)                                       |
| [merge.md](merge.md)                                             | Merge workflow                                         |
| [match-search-merge.md](match-search-merge.md)                   | Combined match / search / merge reference              |
| [compliance-for-healthcare.md](compliance-for-healthcare.md)     | Compliance for healthcare (US HIPAA, UK NHS, …)        |
| [compliance-for-technology.md](compliance-for-technology.md)     | Compliance for technology (ISO, GDPR, …)               |
| [locales.md](locales.md)                                         | Locales, localization, internationalization, languages |
| [observability.md](observability.md)                             | Observability summary (alias for the Loco doc)         |
| [observability-for-rust-loco.md](observability-for-rust-loco.md) | OpenTelemetry, tracing, metrics, logs, spans           |
| [postgresql.md](postgresql.md)                                   | PostgreSQL database, extensions                        |
| [privacy.md](privacy.md)                                         | Data masking, GDPR, consent                            |
| [restful.md](restful.md)                                         | RESTful API guidance                                   |
| [technology.md](technology.md)                                   | Technology stack summary (alias for the Loco doc)      |
| [stack-for-rust-loco.md](stack-for-rust-loco.md)                 | Stack for Rust, Loco (database, search, …)             |
| [loco.md](loco.md)                                               | Loco framework (backend-only conventions)              |
