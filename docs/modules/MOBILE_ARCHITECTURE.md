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

## Round 2 correction — AR-006 / AR-009 / MR-12 / MR-13 (2026-10-12)

**Status: PROPOSED contracts; OPEN business/security policy.** Round 2 audit confirms ADR-MOB-004 and ADR-MOB-009 **already exist**. The earlier missing-file statement is historical/stale registry text, not current repository state. This section refines their contracts without changing APPROVED business decisions or closing the review findings.

### Authority and lifecycle

Mobile is an offline command/evidence producer, **not** an authoritative Sales/Inventory, Finance, Billing, Security or CRM writer. CRM owns visits/routes/outlet relationship; Sales owns commercial order acceptance; Inventory owns physical stock effects; Finance owns collections/GL; Billing owns SaaS charge occurrence, rating and wallet; Security/Multi-Tenant own credential/membership/scope policy; Backend enforces acceptance and idempotency.

**Proposed outbox state machine:** `DRAFT → QUEUED → SENDING → ACCEPTED | REJECTED | CONFLICT | UNKNOWN | RETRYABLE`. `UNKNOWN` means no authoritative acknowledgement (timeout/crash); reconcile by same `operation_id` or status lookup, never mint a new operation for retry. `RETRYABLE → QUEUED` after bounded backoff/jitter and online/auth preconditions; `CONFLICT → NEEDS_REVIEW → QUEUED` only after explicit user/domain-approved resolution and a *new semantic command identity* referencing the original; `REJECTED` is terminal unless corrected into a new command; `ACCEPTED` is immutable acknowledgement, not proof of later stock/GL/Billing completion. A separate `SERVER_EFFECT_PENDING/COMPLETED/FAILED` projection reports downstream business effects by owner. `RESYNC_REQUIRED → RESYNCING → READY` is a **sync cursor** state machine, not a command outcome; preserve queued/unknown commands, tombstones and tenant scope. A successful delta pull cannot turn UNKNOWN into ACCEPTED.

**Durability and dedupe:** atomically persist local business intent + outbox; stable `operation_id` per user intent across retries; `payload_hash` prevents same-key/different-payload silent reuse. Server dedupes scoped command receipt, then domain `business_occurrence_id` prevents duplicate stock/finance/Billing economic effects across event retries; event delivery ID differs from business occurrence ID. Server returns prior result for identical replay; mismatched payload is a conflict/security signal. Do not claim transport exactly-once; target at-most-one authoritative economic posting via durable domain uniqueness and reconciliation.

### Reconnect authorization and device lifecycle

At **every** queued command/evidence/media processing attempt backend validates current authenticated identity, active device/session, tenant, company, branch, membership status, role grants/revocation, assignment and operation/resource scope. Client-supplied scope, stale role cache, GPS and device time are untrusted. Permission or device revoke is a terminal authorization failure for that attempt; never silently reroute command to another tenant/company/branch. Separate refresh/reauth from command retry. Expired token alone may permit reauthentication under Security policy; revoked device must not refresh. No immediate remote wipe guarantee while offline.

**PROPOSED options (Security approval required):** quarantine inaccessible encrypted pending evidence until reviewed vs policy-bound purge after verified server acknowledgement/legal hold; cryptographic key invalidation on logout/revoke detection vs retention for explicitly authorized evidence recovery; offline session TTL and device trust remain OPEN. No automatic upload of quarantined records, no plaintext debug export, no cross-tenant pending queue carryover.

### Delayed usage timestamps and month attribution — AR-009 OPEN

Capture `client_occurred_at` (untrusted), `server_received_at`, `server_accepted_at`, domain-authoritative `occurred_at` when established, `billed_at` (actual charge posting time), and `rated_calendar_month` (Billing decision); record timezone and immutable correction references. **Options, not selected:** A trusted business occurrence month with late corrections; B server acceptance/chargeable transition month; C server receipt month with adjustments. Evaluate calendar boundary, month close, >30-day offline, late cancellation/reversal, discount tiers and negative-month anchoring jointly with Billing. Mobile never chooses the rating month or charges wallet; BIZ-006/007 remain unchanged. AR-010 billable document type/transition also OPEN.

### Workflow × authoritative effect matrix (PROPOSED)

| Offline workflow | Local operation | Server owner and acceptance | Duplicate prevention / failure UI |
|---|---|---|---|
| Customer/outlet | scoped edit/draft | CRM/Party writer decision OPEN | entity version, identity dedup; conflict/manual review |
| Visit/route | check-in/out evidence | CRM verifies visit transition | stable visit occurrence; pending/invalid GPS |
| Sales order | draft/submit intent | Sales accepts; stock **not** decremented on confirmation | command + business occurrence dedup; price/credit conflict |
| Delivery | proof, quantities, media | Sales/Inventory reconcile dispatch; Inventory posts physical movement once | unique dispatch occurrence; proof pending != dispatched |
| Payment evidence | unposted local record | Finance validates and posts once if authorized | unique receipt occurrence; never show settled offline |
| GPS/media | encrypted queued evidence | CRM/Security-controlled evidence validation | evidence ID/checksum; reject revoked scope |
| SaaS usage | **no client charge** | Billing rates authoritative eligible server occurrence | billing unique economic key; delayed month OPEN |

### UX states and recovery

- **Pending / Qurilmada saqlandi:** durable local, not server-confirmed.
- **Syncing / Yuborilmoqda:** in flight; disable accidental duplicate submit while preserving status lookup.
- **Unknown / Tekshirilmoqda:** transport ambiguity; same operation ID reconciliation.
- **Retryable / Qayta uriniladi:** temporary failure with next attempt and manual retry.
- **Conflict / Ko‘rib chiqish kerak:** show safe field differences, responsible role and resolution action; do not auto-merge stock/finance.
- **Failed / Rad etildi:** validation or permission reason and safe correction/appeal path; revoked data is access-blocked.
- **Completed / Server tasdiqladi:** command accepted; if domain posting is asynchronous, show `Business effect pending` until owner confirms completion. No optimistic “paid/dispatched/billed”.

### Failure and adversarial scenario matrix (PROPOSED test design)

| Scenario | Required outcome | Policy OPEN |
|---|---|---|
| Network drops before request | queued survives crash, retry same ID | backoff/batch tuning |
| Network drops after server commit before ack | UNKNOWN, status lookup/retry same ID, no duplicate effect | dedup retention |
| App killed during delta apply | transaction rolls back or cursor+data commit together | cursor expiry threshold |
| Permission/role revoked offline | backend rejects on reconnect; quarantine scope | retention/cleanup authority |
| Device stolen/revoked | server blocks refresh and command/media acceptance; no guaranteed instant offline wipe | offline TTL, remote recovery |
| Token expires during >30-day offline | no presumed validity; reauth/deny on reconnect; drafts preserved only if policy allows | TTL, legal retention |
| Tenant/company/branch switch with pending queue | isolate old scope; never auto-submit in new scope; prompt authorized recovery | export/purge/reassign options |
| Two devices edit same customer | base_version conflict and CRM-owned resolution | merge policy |
| Offline price/stock changed | Sales/Inventory revalidate; no phantom reservation or double dispatch | pricing exceptions |
| Duplicate GPS/media/upload | stable evidence ID/checksum; no duplicate visit/charge | evidence retention |
| Delayed chargeable event crosses month | Billing chooses period after approval; timestamps retained | AR-009/AR-010 |
| Device storage full/media upload interrupted | keep command integrity, report recoverable failure, resume verified chunks | media quota |

**Release evidence (not yet produced):** cross-device/tenant/branch isolation, restart recovery, permission revoke, same-key/different-payload, late event across month, duplicate stock/charge, offline 30+ day and UX-T workflow acceptance. No numeric offline lifetime is approved.
