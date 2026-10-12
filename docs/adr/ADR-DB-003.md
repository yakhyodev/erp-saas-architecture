# ADR-DB-003 — Composite tenant keys + RLS

> Status: **PROPOSED**  
> Scope: Database Architecture  
> Source: [DATABASE_ARCHITECTURE.md](../modules/DATABASE_ARCHITECTURE.md)  
> Tasdiqlangan biznes talablari bilan chalkashtirilmasin. Ushbu texnik qaror hali tasdiqlanmagan.

- **Status:** PROPOSED.
- **Muammo:** cross-tenant FK va query leaks.
- **Variantlar:** app-only filters; RLS-only; composite FK + app authorization + RLS.
- **Tanlov:** layered controls.
- **Sabab:** data-level referential containment va defense-in-depth.
- **Xavf:** RLS/pool context mistakes, index overhead, admin access pitfalls.
- **Ta’sir:** Security, Backend, Integrations, migrations.

## Round 2 MR-11 — tenant isolation acceptance gate (PROPOSED, tests NOT RUN)

Database layer must enforce composite tenant foreign keys, application authorization and fail-closed RLS as defense-in-depth where RLS is selected. Required negative tests: A child cannot reference B parent even when UUID exists; missing tenant context produces no reads/writes; app/worker roles cannot bypass RLS/DDL; pooled connection A->B after commit, rollback, timeout and prepared-statement reuse cannot leak session context; session search_path/tenant switching cannot bypass membership; parallel worker batches and outbox replay always carry explicit tenant; reporting/exports/caches and replica reads do not cross tenant boundaries; enterprise tenant rerouting fails closed. Use transaction-local context rather than session affinity with transaction poolers. Segregate migration/backup/support admin roles from runtime roles, audit privileged operations and SECURITY DEFINER paths. Treat PostgreSQL table-owner RLS bypass and FORCE policy interactions explicitly. Automated test evidence and Security approval OPEN; no passing tests claimed. Database §18.3 is normative proposed test matrix.
