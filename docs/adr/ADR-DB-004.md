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
