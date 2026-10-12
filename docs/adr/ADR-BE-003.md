# ADR-BE-003 — Transaction & idempotency

- **Holat:** PROPOSED
- **Loyiha:** Sales doc copy — ERP SaaS
- **Yo‘nalish:** Backend Architecture
- **Manba:** [BACKEND_ARCHITECTURE.md](../modules/BACKEND_ARCHITECTURE.md)

- **Muammo:** duplicate charge, stock race, half-commit.
- **Variantlar:** best-effort; ACID + outbox + idempotent consumers; distributed 2PC.
- **Tanlov (PROPOSED):** lokal ACID, transactional outbox, inbox/dedup va reversal.
- **Sabab:** auditability va real delivery failurelarga chidamlilik.
- **Xavf:** eventual consistency va outbox backlog; strict transaction boundary.
- **Ta'sir:** Billing, Sales, Inventory, Finance, Database.

## Round 2 addendum — AR-005 (2026-10-12; PROPOSED)
Alternatives: **A** single SalesDocument+StockMovement+GL ACID transaction (historical Database §6 PROPOSED; tight coupling, all-or-nothing), **B** separate authoritative owner ACID commits plus transactional outbox/inbox/retry/compensation (**recommended PROPOSED**, Database §17-compatible), **C** distributed 2PC (reject by default). B persists each owner's state+idempotency response+outbox atomically. Consumer inbox and effect uniqueness prevent double decrement, GL posting or SaaS wallet debit; source remains committed during downstream failure with pending state, retry, DLQ, immutable reversal and reconciliation. Physical stock movement occurs on posted dispatch, not confirmation or invoice; accounting recognition/chargeability OPEN. This does not silently change Database §6; its owner must reconcile older wording. Same-key/different-payload => 409, no mutation. Business occurrence ID is separate from event ID; retries reuse semantic effect key, new envelope does not repeat effect. Risks: eventual consistency, backlog, reordering. Details: [Backend §18.3–18.5](../modules/BACKEND_ARCHITECTURE.md).
