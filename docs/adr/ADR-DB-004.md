# ADR-DB-004 — Atomic document posting and outbox

> Status: **PROPOSED**  
> Scope: Database Architecture  
> Source: [DATABASE_ARCHITECTURE.md](../modules/DATABASE_ARCHITECTURE.md)  
> Tasdiqlangan biznes talablari bilan chalkashtirilmasin. Ushbu texnik qaror hali tasdiqlanmagan.

- **Status:** PROPOSED.
- **Muammo:** concurrent sale/stock/finance mutation va async billing double charge.
- **Variantlar:** distributed transaction; single DB transaction with outbox; fully eventual consistency.
- **Tanlov:** same-DB operational atomicity + transactional outbox for async billing/integrations.
- **Sabab:** correctness without network-held locks or distributed 2PC.
- **Xavf:** eventual usage lag, outbox retries and reconciliation burden.
- **Ta’sir:** Sales, Inventory, Finance, Billing, Integrations.

## Round 2 clarification — R2-TXN-01 / AR-005 (2026-10-12)

**Status: PROPOSED — no business trigger approved.** Original phrase “same-DB operational atomicity” is narrowed to **owner-specific local ACID**. Original broad single SalesDocument+StockMovement+Finance journal ACID recommendation is retained in module §18 as **historical PROPOSED alternative, not selected**; this is not a reversal of any APPROVED decision.

**Problem:** §6 broadly coupled Sales+Inventory+Finance writes while §17 and Sales/Finance authoritative ownership require domain-local posting and outbox.

| Model | Atomicity | Failure/retry and reversal | Decision |
|---|---|---|---|
| A: same-DB cross-domain ACID | Three domain tables and outbox in one shared commit, only if colocated | Rollback before commit; wider contention; Billing/integration still async; linked compensations needed afterward | Historical alternative PROPOSED |
| B: owner-local ACID+outbox | Sales state+outbox, Inventory movement+balance+reservation+outbox, Finance journal+lines+outbox, Billing rating+debit+wallet+outbox separately | Consumer lag/failure handled by durable outbox/inbox, retry/backoff, stable tenant/source/effect dedupe and reconciliation; immutable correction facts | Preferred PROPOSED; domain sign-off OPEN |

**Owner-specific effects:** Sales confirmation never implicitly decrements on-hand; Inventory reservation affects available only; dispatch posts Inventory movement once; invoice never posts stock twice; Finance owns journal and recognition trigger remains OPEN; Billing owns actual usage charge and billability/attribution remains OPEN. Distinct source fact IDs, operation IDs, event IDs, aggregate version and causation IDs must be traceable across delivery. Consumer acknowledgment only after atomic inbox+effect commit; event delivery at least once, effect effectively once, never globally exactly-once transport.

**Verification contracts (NOT RUN):** retry duplicate confirmation, two concurrent reserve/dispatch calls against hot SKU, crash before/after source commit and outbox publish, consumer crash after effect commit before ACK, out-of-order invoice/dispatch, late offline usage, linked partial reverse without rewriting ledger, no duplicate Inventory/Finance/Billing postings, daily reconciliation of source facts vs outbox/inbox/ledgers. Illegal cross-tenant references denied. Cross-domain immediate consistency not promised; no distributed 2PC assumed.

**Impact:** Backend must preserve authoritative command boundaries and versioned events; Sales, Inventory and Finance approve state and GL triggers; Billing decides charge eligibility, policy and calendar attribution; Security enforces tenant isolation. See Database §18, Sales §11, Finance reconciliation, Billing policy and ADR-DB-003/006/007. Approval OPEN.
