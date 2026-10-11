# ADR-BE-007 — Authentication and tenant authorization

- **Holat:** PROPOSED
- **Loyiha:** Sales doc copy — ERP SaaS
- **Yo‘nalish:** Backend Architecture
- **Manba:** [BACKEND_ARCHITECTURE.md](../modules/BACKEND_ARCHITECTURE.md)

- **Muammo:** cross-tenant data exposure, privilege escalation.
- **Variantlar:** client-trusted IDs; app-only checks; verified authn/authz + DB defense-in-depth.
- **Tanlov (PROPOSED):** server-resolved tenant/membership; use-case authorization; DB isolation defense-in-depth.
- **Sabab:** bir necha qatlamda tenant breach oldini olish.
- **Xavf:** RLS/context leak, permission caching stale bo'lishi.
- **Ta'sir:** Multi-Tenant, Security, Database, UX.
