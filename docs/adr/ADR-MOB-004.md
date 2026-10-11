# ADR-MOB-004 — Offline Conflict Handling and Idempotency

- **Status:** PROPOSED
- **Owner:** 09 Mobile & Offline
- **Review:** AR-006
- **Date:** 2026-10-11

## Problem
Offline commands can arrive late, out of order, duplicated, or after server-side entity changes and permission revocation. Mobile must not silently overwrite stock, finance, customer or billing state.

## Alternatives
1. Last-write-wins: simple but can silently lose changes and corrupt business outcomes.
2. Reject every version mismatch: safe but high manual burden.
3. Domain-specific version checks and conflict outcomes with durable idempotency: **PROPOSED**.

## Proposed contract
- Persist business command and outbox record atomically; retain a stable `operation_id` across retries and application restarts.
- Send `tenant_context`, `device_id`, `entity_id`, `base_version`, `payload_schema_version`, and client timestamps for evidence; client timestamps do not authorize operations.
- Backend must authenticate the current principal and recheck tenant membership, branch/assignment scope, active device, and current operation-level permissions **at processing time**, including replay after reconnect. A previously valid offline session does not grandfather revoked rights.
- Idempotency is enforced server-side within the correct tenant and command scope, with durable operation identity and a stable result. Duplicate retry must not create a second order, payment posting, stock movement or SaaS usage event.
- Results: `accepted`, `rejected_validation`, `rejected_permission`, `conflict_version`, `conflict_business`, `retryable`. Transport timeout is **unknown**, not accepted; query/retry the same operation ID.
- `conflict_version`: preserve local draft and server version for domain-specific review. `conflict_business`: e.g., stock shortage or price-policy mismatch, never auto-post. `rejected_permission`: stop retries, quarantine pending evidence, block further access to revoked scope and follow Security-approved cleanup policy.
- Never universally apply last-write-wins; never auto-merge financial, inventory or billing ledger entries. Resolve only under owning domain's approved rules.
- Ordering: honor explicit command dependencies; independent commands may process separately. Failed dependencies must not silently allow dependent posting.
- Audit operation identity, outcome, actor/device/scope, server processing time, and reason codes, without leaking sensitive payloads.

## Rationale
Crash-safe delivery and server-authoritative permission checks prevent duplicate business effects and unsafe overwrites.

## Risks
Conflict backlog, scope changes during long offline periods, idempotency retention policy, delayed media dependencies, and ambiguity in client timestamps.

## Cross-module impact
Security & Permissions owns identity/revocation policy; Multi-Tenant owns scope; Sales/Inventory and Finance own business conflict outcomes; Billing owns charge idempotency; Backend/Database own durable deduplication. No unilateral business approval.

## Open questions
Idempotency retention horizon; permission-revocation local data disposal; price/stock conflict UX and operator authority; server conflict payload redaction; business command ordering. All **OPEN**.

[Mobile Architecture](../modules/MOBILE_ARCHITECTURE.md) · [Open Questions](../../OPEN_QUESTIONS.md)
