# ADR-BE-006 — Event contracts and outbox

- **Holat:** PROPOSED
- **Loyiha:** Sales doc copy — ERP SaaS
- **Yo‘nalish:** Backend Architecture
- **Manba:** [BACKEND_ARCHITECTURE.md](../modules/BACKEND_ARCHITECTURE.md)

- **Muammo:** event publish va DB commit orasida data loss.
- **Variantlar:** direct publish; polling; outbox + relay.
- **Tanlov (PROPOSED):** transactional outbox + versioned events + idempotent consumer.
- **Sabab:** publish failurelarga chidamlilik.
- **Xavf:** duplicate, ordering, schema governance va replay murakkabligi.
- **Ta'sir:** barcha event-producing modullar.

## Round 2 addendum — MR-10 event semantics (2026-10-12; PROPOSED)
Versioned immutable event envelope: event_id, type/schema_version, business_occurrence_id, aggregate_id/revision, tenant/company/branch, occurred_at, committed_at, producer, correlation/causation. **event_id** identifies envelope/delivery identity; **business_occurrence_id** identifies one semantic source fact; downstream owner effects have distinct IDs linked to source occurrence. Aggregate state+outbox+dedupe result in one local transaction. At-least-once relay+bounded backoff/jitter; inbox unique consumer/tenant/event_id AND business-effect unique tenant/effect_type/source_occurrence_id; replay/re-envelope cannot double-post. Same effect key with different fingerprint quarantined; no silent overwrite. Ordering only per aggregate revision, missing predecessors retry/reconcile, no global FIFO assumption; DLQ, poison quarantine and reconciliation required. Transport exactly-once is NOT claimed. Retention, broker and SLOs OPEN. Details: [Backend §18.4](../modules/BACKEND_ARCHITECTURE.md).
