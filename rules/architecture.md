# Architecture

## System Architecture

```
+------------------------------------------------------------------+
|                         Client Layer                              |
|  (Web Apps, Mobile Apps, EHR Systems, Analytics Platforms)        |
+------------------------------+-----------------------------------+
                               |
+------------------------------v-----------------------------------+
|                      API Layer                                    |
|  +-------------------+ +------------------+ +-----------------+  |
|  |   REST API        | |   FHIR R5 API    | |   gRPC API      |  |
|  |   (Axum)          | |   (Axum)         | |   (Tonic)       |  |
|  |   15 endpoints    | |   5 endpoints    | |   (stub)        |  |
|  +-------------------+ +------------------+ +-----------------+  |
|  - OpenAPI/Swagger Documentation (Utoipa)                        |
|  - Validation & Data Quality                                     |
|  - Privacy & Data Masking                                        |
|  - CORS, Error Handling                                          |
+------------------------------+-----------------------------------+
                               |
+------------------------------v-----------------------------------+
|                   Business Logic Layer                            |
|  +---------------+ +----------------+ +-----------------------+  |
|  |   Person     | |   Matching     | |   Search Engine       |  |
|  |  Repository   | |  Algorithms    | |    (Tantivy)          |  |
|  +---------------+ +----------------+ +-----------------------+  |
|  +---------------+ +----------------+ +-----------------------+  |
|  |    Event      | |    Audit       | |   Deduplication       |  |
|  |  Publisher    | |  Log Tracking  | |   Engine              |  |
|  +---------------+ +----------------+ +-----------------------+  |
|  +---------------+ +----------------+                            |
|  |  Validation   | |   Privacy      |                            |
|  |  & Quality    | |   & Masking    |                            |
|  +---------------+ +----------------+                            |
+------------------------------+-----------------------------------+
                               |
         +---------------------+---------------------+
         |                     |                     |
+--------v------+  +-----------v------+  +-----------v--------+
|  PostgreSQL   |  |   Tantivy        |  |  Event Stream      |
|  (SeaORM)     |  |   Search         |  |  (In-Memory)       |
|               |  |   Index          |  |                    |
|  - persons   |  |  11 indexed      |  |  - PersonEvents   |
|  - audit_log  |  |  fields          |  |  - Subscribers     |
|  - 12+ tables |  |                  |  |                    |
+---------------+  +------------------+  +--------------------+
```

## Module Structure

```
src/
├── api/                    # API Layer
│   ├── mod.rs              # ApiResponse, ApiError wrappers
│   ├── rest/               # REST API (Axum)
│   │   ├── mod.rs          # Router setup
│   │   ├── handlers.rs     # 15 endpoint handlers
│   │   ├── routes.rs       # Route organization
│   │   └── state.rs        # AppState (shared state)
│   ├── fhir/               # FHIR R5 API
│   │   ├── mod.rs          # FHIR types, conversions
│   │   ├── handlers.rs     # 5 endpoint handlers
│   │   ├── resources.rs    # FHIR resource converters
│   │   ├── bundle.rs       # FHIR bundle handling
│   │   └── search_parameters.rs
│   └── grpc/               # gRPC API (stub)
│       └── mod.rs
├── models/                 # Domain Models
│   ├── mod.rs              # Shared types (Gender, Address, ContactPoint)
│   ├── identifier.rs       # Identifier, IdentifierType
│   ├── document.rs         # IdentityDocument, DocumentType
│   ├── merge.rs            # MergeRequest, MergeResponse, MergeRecord
│   ├── review_queue.rs     # ReviewQueueItem, BatchDedup request/response
│   ├── consent.rs          # Consent, ConsentType, ConsentStatus
│   └── organization.rs     # Organization
├── db/                     # Database Layer
│   ├── mod.rs              # Connection management
│   ├── models.rs           # SeaORM entities (12 entity modules)
│   ├── schema.rs           # Schema definitions
│   ├── repositories.rs     # PersonRepository trait + SeaORM impl
│   └── audit.rs            # AuditLogRepository
├── matching/               # Matching Engine
│   ├── mod.rs              # Matcher trait, ProbabilisticMatcher
│   ├── algorithms.rs       # All matching algorithms
│   ├── scoring.rs          # Probabilistic + Deterministic scorers
│   └── phonetic.rs         # Soundex implementation
├── search/                 # Search Engine
│   ├── mod.rs              # SearchEngine wrapper
│   ├── index.rs            # PersonIndex (Tantivy)
│   └── query.rs            # Query builder
├── streaming/              # Event Streaming
│   ├── mod.rs              # PersonEvent, EventProducer trait
│   ├── producer.rs         # InMemoryEventPublisher
│   └── consumer.rs         # EventConsumer (stub)
├── validation/             # Data Quality
│   └── mod.rs              # Validation, normalization, standardization
├── privacy/                # Privacy & Compliance
│   └── mod.rs              # Masking, GDPR export, consent checking
├── config/                 # Configuration
│   └── mod.rs              # Config structs, env loading
├── observability/          # Observability
│   ├── mod.rs              # OpenTelemetry setup
│   ├── traces.rs           # Distributed tracing
│   └── metrics.rs          # Metrics collection
├── error.rs                # Error types (11 variants)
└── lib.rs                  # Library root, module declarations
```

## Key Design Patterns

### Trait-Based Abstraction

- `*Repository` — Database operations (SeaORM implementation)
- `*Matcher` — Matching algorithms (Probabilistic, Deterministic)
- `EventProducer` — Event publishing (InMemory, extensible to Kafka/NATS)
- `EventConsumer` — Event consumption (stub)

### Application State

`AppState` in `src/api/rest/state.rs` holds all shared services:

- `db: DatabaseConnection`
- `*_repository: Arc<dyn PersonRepository>`
- `event_publisher: Arc<dyn EventProducer>`
- `audit_log: Arc<AuditLogRepository>`
- `search_engine: Arc<SearchEngine>`
- `matcher: Arc<dyn PersonMatcher>`
- `config: Arc<Config>`

### Data Flow

**Create:** HTTP POST → Validation → Duplicate Detection → Repository INSERT → Search Index → Event Publish → Audit Log → Response

**Match:** HTTP POST → Search Engine (blocking candidates) → Repository GET → Matcher.find_matches → Score + Classify → Response

**Merge:** HTTP POST → Fetch Both → Transfer Data → Update Main → Soft-Delete Duplicate → Update Index → Publish Event → Response

### Database Schema

SeaORM entity modules mapping to PostgreSQL tables:

- `audit_log` — HIPAA-compliant audit trail

### Error Handling

Custom `Error` enum with 11 variants, `thiserror`-derived. `Result<T>` type
alias used throughout. API layer converts errors to `ApiResponse` with
appropriate HTTP status codes.
