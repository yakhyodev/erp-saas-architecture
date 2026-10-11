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
