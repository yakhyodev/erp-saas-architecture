# ADR-DB-008 — Tenant-based optional sharding

> Status: **PROPOSED**  
> Scope: Database Architecture  
> Source: [DATABASE_ARCHITECTURE.md](../modules/DATABASE_ARCHITECTURE.md)  
> Tasdiqlangan biznes talablari bilan chalkashtirilmasin. Ushbu texnik qaror hali tasdiqlanmagan.

- **Status:** PROPOSED.
- **Muammo:** single primary scaling ceiling and enterprise isolation needs.
- **Variantlar:** by-table sharding; by-tenant sharding; immediate microservice DB split.
- **Tanlov:** tenant routing abstraction and optional tenant-level relocation.
- **Sabab:** preserves within-tenant transaction locality.
- **Xavf:** migration cutover risk, global metadata, cross-shard analytics complexity.
- **Ta’sir:** Backend, Multi-Tenant, Infrastructure, Billing.
