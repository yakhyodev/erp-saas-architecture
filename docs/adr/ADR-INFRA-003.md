# ADR-INFRA-003: Queue va cache kiritish tamoyili

- **Status:** PROPOSED
- **Date:** 2026-10-11
- **Owner:** Infrastructure & Scaling
- **Related module:** [INFRA_ARCHITECTURE.md](../modules/INFRA_ARCHITECTURE.md)

## Muammo
Reliable asynchronous work without unnecessary infrastructure.

## Ko‘rib chiqilgan variantlar
Redis and broker immediately; DB-backed durable jobs; synchronous-only execution.

## Tanlangan yechim — PROPOSED
Start DB-backed outbox/durable jobs; introduce dedicated queue or Redis only after measurements.

## Sabab
Fewer components while preserving durable, idempotent billing/inventory effects.

## Xavflar va cheklovlar
DB contention and worker latency may require isolation. Numeric SLO/RPO/RTO and vendor selection are not approved.

## Boshqa modullarga ta’siri
Backend, Billing, Mobile, Integrations.

## Tasdiqlash sharti
Cross-module review and explicit user approval; until then not an approved infrastructure decision.

## Round 2 MR-15 workload gate — 2026-10-12 (PROPOSED)

DB-backed durable outbox/worker jobs remain the simplest candidate until queue saturation or DB contention is measured. Compare Redis/dedicated broker operational cost against DB queue backpressure; never treat cache as source of truth for authorization, inventory or billing. Worker concurrency budget must respect shared DB pool headroom; enforce tenant-aware fairness during month-end and sync bursts, retry-safe occurrence IDs and poison-job quarantine. Load-test oldest-job age, throughput, retries, duplicates and end-to-end latency. Dedicated Kafka/Redis are **not mandatory**. See [Infra §19](../modules/INFRA_ARCHITECTURE.md).
