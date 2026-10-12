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

## Round 2 addendum — AR-013 / MR-11 identity and authorization (2026-10-12; PROPOSED)
Candidate **sole writers**: Identity/Auth credentials+sessions+device trust; Multi-Tenant TenantMembership status/revision and company/branch graph; Security RoleAssignment/grants/policy, Security PermissionEvaluation behavior. Backend enforces at **every API, worker, scheduler, integration, export, search, admin and offline sync command**: validate principal and current revocation, resolve tenant/branch server-side, fresh membership, permission+object scope; fail closed. UI, token claims or cached role alone never authorize. Cross-tenant/branch references denied; RLS transaction-local tenant context and pool reset provide defense in depth, not substitute. Revoke invalidates future work including queued mobile commands, committed effects use explicit reversal. Old Security/Billing 'Identity owns memberships' wording **STILL CONFLICTING PROPOSED**, not silently approved. Revoke freshness, role inheritance, exact canonical ownership approval OPEN. Negative tenant isolation and pooling tests NOT RUN. Details: [Backend §18.1, §18.6](../modules/BACKEND_ARCHITECTURE.md).
