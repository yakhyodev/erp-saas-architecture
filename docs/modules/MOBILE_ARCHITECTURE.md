# MOBILE_ARCHITECTURE.md

Status: **PROPOSED** | Version: 1.0 | 2026-10-11

## Approved context
APPROVED business rules: Hybrid Prepaid SaaS; low-cost Seat + document-based Business Usage + Resource Usage; 1 credit = 10 UZS; progressive discounts and enterprise contracts; real Seat/Usage costs may create negative balance for one calendar month without estimated credit limit; unified company/branch SaaS; uz/en/ru plus hidden future Uzbek alphabet.

## Technical status
Flutter, Go, PostgreSQL and Next.js remain **PROPOSED**, not approved. Offline-first and modular monolith also remain proposed.

## Boundaries
Mobile owns local UX, encrypted storage, sync, device lifecycle, GPS and media. Backend owns authorization and final Sales/Inventory, Finance and Billing decisions.

## Offline mode
Online/degraded/offline states. Assigned customers, catalog, prices, routes, visits, order drafts, returns, delivery proof, GPS, media and unposted payment evidence are available locally. No offline final stock reservation, ledger posting, invoice or SaaS charge.

## Local storage and scoping
Tenant > company > branch > user > role > assignment > device. Suggested entities: local_customers, products, prices, routes, visits, orders, order_items, payments, stock_snapshot, sync_outbox, sync_cursor, sync_conflicts, media_queue. Encrypt DB and media, secure keys in OS storage, invalidate and reseed on scope changes.

## Sync
Atomic local command+outbox. Push stable operation_id, device_id, entity_id, base_version, client_created_at and payload_schema_version. Server validates tenant/permissions/business rules and returns accepted/rejected/conflict. Pull paged delta with scoped opaque cursor and tombstones. Apply delta and cursor checkpoint atomically. Retry temporary errors with backoff+jitter; no infinite auth retry. Safe resync preserves pending commands.

## Conflict policy
Never universal last-write-wins. Customer edit: version check and selective merge/manual review. Price changed: server repricing/exception. Insufficient stock: order exception. Duplicate payment/order: server idempotency. Revoked rights: reject and audit.

## Agent and expediter workflow
Agent: Today > route > customer > visit > order > pending > server acknowledgement. Expediter: manifest > delivery > photo/signature/GPS > returns/payment evidence > server reconciliation. Show pending/accepted/rejected/conflict/awaiting evidence. Key actions discoverable in 2–3 steps where practical.

## GPS
Capture visit check-in/out and delivery proof with accuracy, device time and server receive time. Optional route tracking only under explicit policy. Flag spoofing/low accuracy; never invent missing coordinates. Consent and retention unresolved.

## Image upload
Capture > validate > compress/minimize metadata > encrypt locally > durable media queue > retry/resumable upload > checksum and server acknowledgement > attachment. Never delete unacknowledged media; define orphan cleanup.

## Auth
Short access token, protected refresh rotation, bounded offline session, device registration/revocation, server-enforced authorization. Immediate remote wipe while offline cannot be guaranteed.

## Stock and billing
Stock snapshots are informational, not global reservations. Physical delivery manifest requires reconciliation. Offline device does not debit SaaS wallet. Server deduplicates usage events; occurred_at and received_at differ. Billing period for delayed sync remains OPEN QUESTION. Approved negative-balance policy unchanged.

## Observability and performance
Indexed local search, paged catalog, adaptive batches, battery-aware GPS, queue depth, sync latency, conflict/rejection metrics, crash-safe recovery. No unapproved numeric SLA.

## ADR registry
- [ADR-MOB-001](../adr/ADR-MOB-001.md) — Offline-first model — PROPOSED
- [ADR-MOB-002](../adr/ADR-MOB-002.md) — Scoped encrypted local storage — PROPOSED
- [ADR-MOB-003](../adr/ADR-MOB-003.md) — Delta synchronization — PROPOSED
- [ADR-MOB-004](../adr/ADR-MOB-004.md) — Conflict resolution and idempotency — PROPOSED
- [ADR-MOB-005](../adr/ADR-MOB-005.md) — Agent and expediter workflow — PROPOSED
- [ADR-MOB-006](../adr/ADR-MOB-006.md) — GPS evidence and privacy — PROPOSED
- [ADR-MOB-007](../adr/ADR-MOB-007.md) — Media upload lifecycle — PROPOSED
- [ADR-MOB-008](../adr/ADR-MOB-008.md) — Offline stock and billing consistency — PROPOSED
- [ADR-MOB-009](../adr/ADR-MOB-009.md) — Mobile authentication and device security — PROPOSED

## Open questions
Offline session TTL; payment evidence restrictions; price version rules; manifest reconciliation; GPS/media retention; late-arriving usage billing period; sync SLO.

## Codex specification — architecture only
Specify Flutter candidate component boundaries, local schema, sync push/pull contracts, command state machine, idempotency, auth/device security, GPS/media lifecycle, failure recovery, telemetry and test matrix. Do not generate Dart, SQL or backend code. Do not promote proposals to APPROVED.

## Architecture Review reconciliation — AR-006 and AR-009 (PROPOSED)

- AR-006: [ADR-MOB-004](../adr/ADR-MOB-004.md) documents conflict handling and durable idempotency; [ADR-MOB-009](../adr/ADR-MOB-009.md) documents authentication, offline device security and permission revocation. Both remain **PROPOSED**.
- Every queued command is reauthenticated and reauthorized against current tenant membership, branch/assignment, device status and operation permissions **when the backend processes it**. Offline access never guarantees future server acceptance. Revoked scope: reject, audit, quarantine/clean up local data under Security policy.
- Conflict outcomes: accepted; rejected_validation; rejected_permission; conflict_version; conflict_business; retryable. Timeout is unknown, not accepted. Preserve the stable operation_id for safe retries; no universal last-write-wins or automatic stock/finance/billing posting.
- **AR-009 — OPEN QUESTION, joint owner Mobile + Billing:** preserve distinct `client_occurred_at` (untrusted device timestamp), `server_received_at`, `server_accepted_at` (if accepted), server-generated chargeable lifecycle timestamp and the eventual `rated_calendar_month`. Keep original timestamps for audit; do not infer the billing month from mobile clock.
- **Attribution options, none approved:** (A) month of the domain-approved business occurrence, subject to trusted server evidence and late-event adjustment; (B) month of server acceptance/chargeable lifecycle transition; (C) month of server receipt with correction/reversal policy. Compare month boundaries, timezone, offline delays, duplicates, reversals, month close and backdated adjustments with Billing.
- Billing owns charge trigger (AR-010), calendar-month/timezone attribution, rating, discounts, corrections and one-month negative balance accounting. Mobile only delivers deduplicated evidence/commands and does not charge locally. No option is approved or selected here.
- Relevant central review registers: [DECISIONS.md](../../DECISIONS.md) and [OPEN_QUESTIONS.md](../../OPEN_QUESTIONS.md). The central review items remain open until owners formally close them.
