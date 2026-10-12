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

## Round 2 current-main clarification — Database §18.2 (2026-10-12; PROPOSED)

Database Architecture §18.2 has **already documented** the historical same-DB cross-domain ACID option A and the preferred owner-specific ACID+transactional outbox option B. Older Database §6 combined Sales/Inventory/Finance posting language is a preserved historical PROPOSED alternative, **not** executable authority to cross-write ledgers. Backend recommends **B (PROPOSED)** for each domain's owner-state+idempotency+outbox atomic unit, independent Inventory stock posting, Finance GL journal posting and Billing economic charge, with at-least-once delivery, inbox/business-effect dedup, retries/reconciliation and immutable reversals. No single global commit guarantee or accounting/charge trigger approval is implied. Sign-off on GL/COGS/AR revenue recognition, invoice timing, stock negative/reservation policies and billing chargeable transition remains **OPEN**. Cross-reference [Backend §19](../modules/BACKEND_ARCHITECTURE.md) and [Database §18.2](../modules/DATABASE_ARCHITECTURE.md).
