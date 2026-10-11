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
