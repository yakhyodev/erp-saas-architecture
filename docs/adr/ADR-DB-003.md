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
