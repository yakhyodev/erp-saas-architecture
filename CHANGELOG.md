# CHANGELOG.md

- **Version:** 0.1 (2026-10-11)
- **Scope:** central cross-module registry; source main and user-provided 14-module audit
- **Governance:** APPROVED user business rules are binding; proposals are not approvals; missing provenance remains unresolved.
- **Source:** [Master Architecture](https://github.com/yakhyodev/erp-saas-architecture/blob/main/docs/modules/MASTER_ARCHITECTURE.md)

## 2026-10-11 — v0.1 central registry bootstrap (documentation only)

- Created versioned central `REQUIREMENTS.md`, `DECISIONS.md`, `MODULE_REGISTRY.md`, `OPEN_QUESTIONS.md`, `CHANGELOG.md` on existing `main` branch.
- Distinguishes user-approved business requirements from unapproved architectural proposals.
- Captures 13 audit reconciliation issues (AR-001..013) as OPEN QUESTION, with responsible chat and proposed alternatives.
- Preserves FIN-001 reported APPROVED status without fabricating approval provenance (AR-004).
- No product code, price schedule, limits, new enterprise terms, other module ADRs, repository metadata or branch settings changed.
- Source audit snapshot: 108 Markdown files / 90 ADRs, at baseline `016573c8426ed3ea3b1eacbd0dc81adb67861220`; counts were not independently recounted here.
- See per-file GitHub commits for write provenance; this entry is not proof of user approval.

## Next steps (OPEN QUESTION)

Resolve AR-001..013 with domain owners and explicit user sign-off, then update root registries and module-specific ADRs without overwriting history.


## 2026-10-12 — v0.2 Round 2 governance correction

- Source: [Round 2 audit](docs/reviews/ARCHITECTURE_REVIEW_ROUND_2.md), historical evidence HEAD `184b32f7b3dcc9ce8d329ca2dfb7895f06cae905` (97 ADR, 17 modules). Audit snapshot is not the same as current main HEAD.
- AR-006 corrected: ADR-MOB-004/009 **exist**; mobile security policies remain **PROPOSED / OPEN**.
- AR-007 corrected: five root registries **exist**; synchronization is ongoing.
- `OPEN_QUESTIONS` added MR-01..20 statuses, owners, dependencies, decision types, evidence and next actions; P0/P1 queue proposed.
- `DECISIONS` added 97-row ADR status/provenance index without duplicating ADR descriptions; FIN-001 existing APPROVED label maintained with **UNVERIFIED approval provenance**.
- `MODULE_REGISTRY` added explicit PROPOSED Catalog/Product/Pricing, Party/Payer, Identity/TenantMembership/RoleAssignment and dependency boundaries with unresolved alternative owners.
- `REQUIREMENTS` keeps whole-ERP usability as user-approved goal while UX targets remain unmeasured/unapproved; HR, Manufacturing, Purchasing, Reporting MVP inclusion/exclusion OPEN.
- `docs/modules/MASTER_ARCHITECTURE.md` preserves historical content and links one Round 2 dependency map plus source audit.
- No implementation, no code, no other domain documents, no newly approved technical architecture.
