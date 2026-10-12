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

## Round 2 refinement — AR-006 / MR-12 / MR-13 (PROPOSED)

**This ADR already exists.** The review's physical-file gap is resolved; its security and conflict decisions remain unapproved.

### Explicit command and sync contract

Outbox: DRAFT → QUEUED → SENDING → ACCEPTED / REJECTED / CONFLICT / UNKNOWN / RETRYABLE. Timeout/ack loss = UNKNOWN; query status or retry **same** operation_id and unchanged payload hash. Same operation_id + changed payload must be rejected as a conflict, not accepted as another business operation. Retryable temporary failures use backoff/jitter; authorization failure is not a transient retry. A user-resolved changed business intent creates a **new** operation_id linked to the old one. Command ACK != downstream physical stock movement, GL posting or SaaS charge.

Delta cursor: READY → PULLING → APPLYING → READY; invalid cursor or scope revision → RESYNC_REQUIRED → RESYNCING → READY. Preserve pending/unknown commands and commit cursor atomically with data/tombstones. Server deduplication uses tenant-scoped command identity; domain posting uses canonical business_occurrence_id, distinct from delivery event_id. No claim of exactly-once network delivery.

### Server replay and conflict matrix

| Conflict | Server behavior | Client recovery |
|---|---|---|
| Version changed | compare expected aggregate revision; no blind overwrite | show differences; CRM/Sales owner resolves |
| Price/credit/stock changed | Sales/Inventory authoritative validation; no offline stock effect | pending business review |
| Same key, same payload | return prior accepted/rejected outcome | reconcile UNKNOWN |
| Same key, different payload | explicit idempotency conflict + audit | create new intent after review |
| Permission/membership/role revoked | reject and audit at processing time | quarantine, block unauthorized access |
| Device revoked | reject command/media and token refresh | reauth only through Security-approved recovery |
| Cross-tenant/branch mismatch | deny; no automatic retarget | explicit scope recovery |
| Duplicate visit/delivery/payment/charge | stable domain occurrence dedup | show single authoritative result |

On **every** queued attempt verify current identity, tenant/company/branch, membership, role grant, device, assignment and operation/resource scope. Revoke invalidates cached authority. Server accepts only authenticated verified context, not client tenant claims.

**OPEN:** dedup retention, field merge authorization, conflict payload minimization, manual review ownership, domain occurrence uniqueness, dependency ordering and operational backpressure. Billing month policy remains with Billing (AR-009); no business approval is inferred.
