# ADR-DB-001 — PostgreSQL transactional store

> Status: **PROPOSED**  
> Scope: Database Architecture  
> Source: [DATABASE_ARCHITECTURE.md](../modules/DATABASE_ARCHITECTURE.md)  
> Tasdiqlangan biznes talablari bilan chalkashtirilmasin. Ushbu texnik qaror hali tasdiqlanmagan.

- **Status:** PROPOSED.
- **Muammo:** OLTP ACID, murakkab FK va reporting talablarini bitta ishonchli persistence layerda bajarish.
- **Variantlar:** PostgreSQL; MySQL; distributed SQL.
- **Tanlov:** PostgreSQL (taklif).
- **Sabab:** mature transactional integrity, RLS, indexing, WAL/PITR, replication, partition imkoniyatlari; boshlang‘ich operatsion sodda.
- **Xavf:** primary write bottleneck, vacuum/tuning va operator tajribasiga bog‘liqlik.
- **Ta’sir:** Backend, DevOps, all domain modules.
