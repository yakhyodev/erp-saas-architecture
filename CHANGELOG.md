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


## 2026-10-12 — v0.3 final cross-module registry synchronization

- Compared current `main` source [current-main sync baseline](https://github.com/yakhyodev/erp-saas-architecture/commit/f26d4587dcad20f4bf7e8404e27cdcaeace9a355) to prior Master write `6f6c7e9bb04feaf9a586bfcc964110eef6a79084`: **83 commits ahead**, changed module/ADR documents, **3 new ADRs** (ADR-SI-014, ADR-SI-015, FIN-003).
- Extended central decision status/provenance index from 97 historical known ADRs to 100 known ADRs: 99 PROPOSED and FIN-001 APPROVED-labelled/provenance-UNVERIFIED. No ADR or user approval rewritten.
- Updated AR-001…AR-013 and MR-01…MR-20 current status tracking; corrected DB transaction conflict to **documented historical alternate vs preferred proposal**, still approval OPEN.
- Recorded five Billing §21 user-selected **SELECTED_PENDING_APPROVAL** candidates, without falsely labeling them approved; no new price, limit, duration interpretation or resource rate finalized.
- Updated authoritative writer/dependency proposals (Catalog, Pricing, Sales, Inventory, Purchasing, Party, Finance debtor, Billing payer, Identity, Membership, Roles and PermissionEvaluation).
- Linked restored SalesDoc V2 provider compatibility appendix: documentary evidence is not live API verification.
- AR-006/007 documentation exists, while Mobile/Security and governance sign-off are still OPEN. UX usability invariant maintained; MVP HR/Manufacturing/Purchasing/Reporting OPEN.
- Historical audits and other chat modules/ADRs intentionally not modified. Codex readiness not reassessed; no code written.

---

## 2026-10-12 — v0.4 user-approved MVP Company-scoped Billing and shared Resource Usage

- Added [ADR-BILL-011](docs/adr/ADR-BILL-011.md), the 101st ADR, with overall PROPOSED status and separated APPROVED_BY_USER business subset from operational/technical OPEN decisions.
- User explicitly approved individual Company LegalEntity/BillingAccount/Wallet (1:1:1:1), that LegalEntity as SaaS debtor, third-party authorized funding without liability transfer, Seat once per user per Company (no Branch/device duplication), and one predesignated responsible Company for Tenant-shared measured Resource Usage.
- Appended scope-limited synchronization notes to Billing, Multi-Tenant, Finance and Master modules, plus DECISIONS, REQUIREMENTS, MODULE_REGISTRY and OPEN_QUESTIONS; previous proposal alternatives remain historical.
- Two-step shared assignment, prospective change, immutable historical charges, pending attribution and dual-role permission tests are proposed defaults; exact financial cutoffs, price, tax, refund, Seat cycle and technical ADR approval remain OPEN. No code, runtime test, false readiness or old audit rewrite.

---

## 2026-10-12 — v0.5 Seat lifecycle ADR-BILL-012 (design only)

- Added [ADR-BILL-012](docs/adr/ADR-BILL-012.md) (102nd ADR) describing candidate activated Seat lifecycle, calendar-day proration variants, unique economic occurrence per user×Company/day, event-version ordering and linked reversals.
- Existing ADR-BILL-011 explicit user-approved per-Company Seat scope preserved; activation/suspend/day cutoff/pricing/refund **not automatically approved**.
- Reconciled Billing, ADR-BILL-010 and Multi-Tenant plus central decision, requirements, registry, open questions and Master architecture documentation. Original audit snapshots retained; no product code/tests/prices invented.
