# ADR-DB-009 — Expand/contract database evolution

> Status: **PROPOSED**  
> Scope: Database Architecture  
> Source: [DATABASE_ARCHITECTURE.md](../modules/DATABASE_ARCHITECTURE.md)  
> Tasdiqlangan biznes talablari bilan chalkashtirilmasin. Ushbu texnik qaror hali tasdiqlanmagan.

- **Status:** PROPOSED.
- **Muammo:** migration without prolonged platform outage.
- **Variantlar:** big-bang DDL release; expand/backfill/verify/contract.
- **Tanlov:** backwards-compatible staged migrations.
- **Sabab:** continuous service and rollback compatibility.
- **Xavf:** temporarily duplicated representations, longer rollout.
- **Ta’sir:** Backend, QA, DevOps, enterprise fleet.
