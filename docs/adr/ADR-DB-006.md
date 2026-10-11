# ADR-DB-006 — Tenant-aware indexes and measured partitioning

> Status: **PROPOSED**  
> Scope: Database Architecture  
> Source: [DATABASE_ARCHITECTURE.md](../modules/DATABASE_ARCHITECTURE.md)  
> Tasdiqlangan biznes talablari bilan chalkashtirilmasin. Ushbu texnik qaror hali tasdiqlanmagan.

- **Status:** PROPOSED.
- **Muammo:** mixed tenant OLTP hot paths and long event histories.
- **Variantlar:** blanket indexes/partition all; targeted composite indexes + evidence-based time partitioning.
- **Tanlov:** targeted indexes, keyset pagination, measured partitioning.
- **Sabab:** less write amplification and schema complexity before justified.
- **Xavf:** growing table skew; partition uniqueness traps.
- **Ta’sir:** Reporting, Billing, Inventory, Ops.
