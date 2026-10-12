# MOBILE_OFFLINE.md

Status: **PROPOSED** | 2026-10-11

Companion to [MOBILE_ARCHITECTURE.md](MOBILE_ARCHITECTURE.md). No separate competing decisions.

## Approved context
APPROVED business rules: Hybrid Prepaid SaaS; low-cost Seat + document-based Business Usage + Resource Usage; 1 credit = 10 UZS; progressive discounts and enterprise contracts; real Seat/Usage costs may create negative balance for one calendar month without estimated credit limit; unified company/branch SaaS; uz/en/ru plus hidden future Uzbek alphabet.

## Offline-first rules
Only assigned scoped data is stored. Local commands and durable outbox are atomic. Lifecycle: draft > queued > sending > accepted/rejected/conflict. Timeouts do not imply acceptance; stable operation_id enables safe retries.

## Delta sync and recovery
Push commands with dependency order; pull paged delta, tombstones and scope-bound cursor; apply changes and checkpoint in one transaction. Expired cursor triggers reseed without losing pending operations. Background execution is not guaranteed.

## Consistency
Stock snapshot is informational. Payment is unposted evidence. Central server authorizes and finalizes inventory, financial ledger and SaaS billing exactly once. Late usage period is unresolved; approved minus-balance rule remains unchanged.

## Security and evidence
Encrypted local DB and media; OS key storage; bounded offline session; server-side revocation. GPS may be absent, never fabricated. Media uses checksums and acknowledgement before cleanup.

## ADRs
See [Mobile ADR registry](MOBILE_ARCHITECTURE.md#adr-registry).

## AR-006 — conflict and revoked-permission recheck (PROPOSED)

See [ADR-MOB-004](../adr/ADR-MOB-004.md) and [ADR-MOB-009](../adr/ADR-MOB-009.md). On every queued-command processing attempt, the server must recheck active identity, tenant membership, branch/assignment, device status and operation permission. A prior offline authorization does not override revocation. Responses distinguish version conflict, business conflict, validation rejection, permission rejection and retryable error. Timeout does not mean success; reuse stable operation_id and retain draft for review. No last-write-wins for ledger/stock/billing.

## AR-009 — delayed usage month attribution (OPEN QUESTION)

Preserve `client_occurred_at` (untrusted), `server_received_at`, `server_accepted_at`, trusted chargeable lifecycle time when defined, and final `rated_calendar_month`. Possible billing attribution: occurrence month with auditable corrections; server acceptance/chargeable transition month; server receipt month with correction policy. **None approved.** Billing and Sales must settle charge trigger (AR-010), timezone, month closing, late events, duplicate/replay, reversal and adjustment rules. Mobile does not rate usage or change approved one-calendar-month negative balance policy.

## Round 2 operational contract — AR-006 / AR-009 / MR-12 / MR-13 (PROPOSED)

**Existing ADRs:** [ADR-MOB-004](../adr/ADR-MOB-004.md) and [ADR-MOB-009](../adr/ADR-MOB-009.md) exist; their earlier “missing” description is stale historical tracking.

### Two independent state machines

- **Outbox command:** `DRAFT → QUEUED → SENDING → ACCEPTED | REJECTED | CONFLICT | UNKNOWN | RETRYABLE`. UNKNOWN = timeout/ack lost, not rejection or acceptance; query/retry same operation_id. RETRYABLE → QUEUED with backoff. CONFLICT → NEEDS_REVIEW; corrected semantic intent uses new operation_id and original reference. REJECTED is terminal unless a new authorized command is created. ACCEPTED confirms only command acceptance, not asynchronous dispatch/GL/Billing.
- **Delta cursor:** `READY → PULLING → APPLYING → READY` or `RESYNC_REQUIRED → RESYNCING → READY`. Scope mismatch/cursor expiry forces reseed; local pending/unknown intents are never silently deleted. Data+tombstones+cursor commit atomically.
- **Economic effect:** domain owner separately reports PENDING/COMPLETED/FAILED; mobile does not infer stock or charge from outbox ACK.

### Authorization and local protection

On every server replay: authenticate principal/session/device; independently resolve tenant, company, branch, membership, active role/permission, assignment and target resource. Reject revoked device/permission, never replay across tenants. Encrypted DB/media and OS-backed keys; offline revoke cannot guarantee instant wipe. Quarantine vs encrypted retention vs cryptographic deletion remain Security-owned OPEN options, including >30-day offline and device theft.

### Event clocks and Billing alignment

`client_occurred_at` untrusted; `server_received_at`, `server_accepted_at`, authoritative `occurred_at`, `billed_at` and `rated_calendar_month` are distinct. Occurrence-month / acceptance-month / receipt-month alternatives remain OPEN with Billing. Timezone, month close, reversals and negative-month anchor cannot be selected by Mobile. Deduplicate domain business occurrence before any Billing rating; no local wallet mutation.

### Offline UX

Pending, syncing, unknown, retryable, conflict, failed, completed and downstream effect pending must be distinguishable by text/icon (not color only), with accessible reason and recovery. Customer/visit/order/delivery/GPS/media evidence retains separate server status and scoped permission checks.

### Test scenarios

Network before/after commit; crash during outbox and delta; permission revoked; stolen device; expired token; 30+ day offline (test case **not approved TTL**); cross-tenant/company/branch switch; duplicate order/dispatch/payment/charge; GPS/media interruption; month boundary delayed event. Expected: no silent loss, no unauthorized replay, no duplicate economic effect, no false completed state.
