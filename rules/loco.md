# Loco

https://loco.rs/

The Main X Index service crates use Loco's backend conventions
(config, hooks, workers, REST API) without its view/template tier —
they are backend-only services. There is no Tera, HTMX, Alpine, or
Lily Design System integration.

## Stack

Use:

- Tantivy search engine
- OpenTelemetry metrics
- Prometheus metrics
- Podman containerization with Debian 13 slim

## Background jobs

https://loco.rs/docs/processing/workers/

- Use Postgres-backed background jobs
- NOT SQLite-backed background jobs (bg_sqlt)
- NOT Redis-backed background jobs (bg_redis)

Config:

```
queue:
  kind: Postgres
  uri: "TODO"
  dangerously_flush: false
  num_workers: 2
```
