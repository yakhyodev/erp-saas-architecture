# Codex Start Here — ERP SaaS architecture-to-implementation handoff

- **Status:** `HANDOFF_PACKAGE_PREPARED / CODE_IMPLEMENTATION_NOT_AUTHORIZED`
- **Updated:** 2026-10-12; **Repository:** `yakhyodev/erp-saas-architecture`; read latest `main` HEAD on each run.
- **Owner:** 00 Master, scope from user-delegated planning; never infer `APPROVED` from an assistant recommendation or a historical `PROPOSED` ADR.
- **Priority:** protect real money, tenant data, financial/stock correctness and user experience rather than maximize complexity.

## 0. Read in order before any development decision
1. `DECISIONS.md`, `REQUIREMENTS.md`, `MODULE_REGISTRY.md`, `OPEN_QUESTIONS.md`, `CHANGELOG.md`.
2. [Delegated coherent MVP baseline](../adr/ADR-010-mvp-delegated-baseline.md).
3. [Company Scoped Billing and legal debtor ADR-BILL-011](../adr/ADR-BILL-011.md), [Seat ADR-BILL-012](../adr/ADR-BILL-012.md), [Negative period ADR-BILL-013](../adr/ADR-BILL-013.md), [Chargeable Docs/Tiers ADR-BILL-014](../adr/ADR-BILL-014.md).
4. `docs/modules/MASTER_ARCHITECTURE.md` and `docs/modules/BILLING_ARCHITECTURE.md`, `MULTI_TENANT_ARCHITECTURE.md`, `DATABASE_ARCHITECTURE.md`, `BACKEND_ARCHITECTURE.md`, `SALES_INVENTORY.md`, `FINANCE_ARCHITECTURE.md`, `SECURITY_ARCHITECTURE.md`, `CRM_DISTRIBUTION.md`, `UX_ARCHITECTURE.md`, `MOBILE_ARCHITECTURE.md`, `INFRA_ARCHITECTURE.md`, `INTEGRATION_ARCHITECTURE.md`, `LOCALIZATION_ARCHITECTURE.md`.
5. [Historical Round 3 evidence](../reviews/ARCHITECTURE_REVIEW_ROUND_3.md), frozen to its prior HEAD: treat as historical audit and validate new deltas independently.
6. [MVP_IMPLEMENTATION_BACKLOG.md](MVP_IMPLEMENTATION_BACKLOG.md) and [REMAINING_GATES.md](REMAINING_GATES.md).

## 1. What's truly approved vs only selected

**Explicit user-approved:** Hybrid Prepaid; cheap Seat+Business Usage+Resource Usage; per-document base + credits beyond measured threshold; 1 credit=10 UZS; progressive discounts/enterprise contracts; real charges continue into negative for one calendar month; **no estimated/forecast negative credit cap**; company/branches in single SaaS; `uz/en/ru` public+future Uzbek alphabet hidden; exceptional simple UX goal; MVP separate Company→distinct LegalEntity→BillingAccount→Wallet, legal debtor = Company's LegalEntity; authorized third-party funder not debtor; user×Company Seat once across branches/devices; one accepted responsible Company for measured Tenant-shared Resource Usage.

**Design-selected under user delegation, not formal commercial/legal approval:** Go+Postgres+Next.js+Flutter, modular monolith, shared-schema transaction-safe tenant isolation, owner-local ACID+outbox, canonical shared Party/Catalog, stock/GL event matrix, Stage A for internal pilot, activated/day-proration Seat, anniversary-clamp negative grace, configured finalized docs only, marginal tiers, one responsible Company accepting shared cost, MVP domain shortlist. See ADR-010/013/014.

**Externally unknowable and NOT TO FILL WITH DEFAULT FEES:** legal jurisdiction/tax interpretation, statutory invoice/functional currency choices, customer tariff and tier amounts, RPO/RTO contract, provider credentials/LIVE_VERIFIED outcomes, privacy retention days and consent basis, completed security/performance/UX evidence. Keep these behind explicit configurable disabled gates rather than quietly guess.

## 2. Hard invariants for Codex

- One canonical authoritative writer per entity. Source never infers `tenant_id == company_id == billing_account_id`. Company legal liability and real payer distinct; no cross-wallet transfers without separate approved transaction.
- `user_id × company_id` Seat economic identity. Membership/branch/device counts not Seat count. No activated/proration price posting without approved priced policy.
- Each business operation has stable `business_occurrence_id` **distinct** from `event_id`. Owner-local ACID + outbox, inbox dedup; retries never double-post stock or charge or GL. `same-key+different-payload` conflict.
- Sales confirmation is **not** physical stock decrement; dispatch/accepted receipt once; invoice does **not** decrement stock again. Finance GL independently posts only under configured compliant recognition policy; SaaS Billing ledger never merged with tenant ERP GL/AR/AP.
- Wallet charges must be actual, tariff-authorized and attributable; no estimated debt limit. Negative period anniversary-clamp is proposed; no unapproved auto suspension.
- AuthZ on server per tenant/company/branch/resource/use-case; pooled connection RLS/tenant context negative tests; cache/search/count/facet/export/object store worker isolation. Client/offline no authority to rate wallet or post ERP GL or stock.
- External SalesDoc adapter evidence DOCUMENTED != LIVE_VERIFIED; no production outbound sync or irreversible migration until verified cutover.
- Locale/search only aliases, never mutate canonical SKU, INN, document number, ledger amount, IDs. Future alphabet default hidden/OFF.
- Product UX should be faster to understand than Sales Doctor; demonstrate by comparative task-based research, never advertise unmeasured superiority.

## 3. Safe phase boundaries

**Current phase (this repository, now):** architecture Markdown/ADRs, dependency graph, decision/acceptance matrix and Codex instructions ONLY. **No product source code, SQL migrations, infrastructure provisioning, real API calls, payment execution or production deployment authorized yet.**

**When user later explicitly authorizes application implementation:** Codex first validates the readiness and external gates in [REMAINING_GATES.md](REMAINING_GATES.md), asks for missing *essential secrets/legal inputs* only, identifies app source repository/directory, and executes narrowly scoped increments from [MVP_IMPLEMENTATION_BACKLOG.md](MVP_IMPLEMENTATION_BACKLOG.md). Stage 0 can include harmless setup/spec infrastructure without enabling economic effects; financial/ERP production features must remain disabled until price, law, tenant isolation and test gates pass. No silence-as-consent.

**No fabricated green checks:** design `TEST_DEFINED` differs from `TEST_EXECUTED_PASS`. Architecture can be a useful build blueprint while `PRODUCTION_READY=NO`.

## 4. Quality acceptance checklist for each later code PR
- Explicit owning domain/API/event contract and cross-module readers; tenant/company provenance.
- Threat model, authRLS negative tests (tenant A vs B, Branch A vs B), same-key-different-payload and concurrent replay tests; permission revoke/offline replay tests.
- DB transaction/reversal semantics and reconciliation; no order confirm double decrement, no invoice re-dispatch, no duplicate Wallet charging.
- Source schema version migrations and backward compatibility; structured error codes across web/mobile/locale.
- Performance at measured pilot workload; resource budgets and recovery evidence.
- UX 20 task contracts with loading/empty/error/offline/unknown states and accessible uz/en/ru locale bundles; task acceptance/failure outcomes.
- No changed `APPROVED` rules without explicit user authorization; ADR decision record updated with evidence and dates.

## 5. Copyable future Codex kickoff prompt (NOT for execution now)

> Read `docs/handoff/CODEX_START_HERE.md`, `MVP_IMPLEMENTATION_BACKLOG.md`, `REMAINING_GATES.md` and all linked APPROVED business rules and selected architecture ADRs at the latest `main`. First produce a concise implementation readiness and risk report, dependency-ordered backlog, all remaining external prerequisites, and a proposed Stage 0 plan. Do not change files, create code, run migrations or deploy until I explicitly authorize a named implementation stage and target app repository. Treat technical defaults as DESIGN_SELECTED rather than user-approved regulatory or monetary rules; never invent rates, limits, test outcomes or provider verification.

## 6. Deliverable naming / release state

`ARCHITECTURE_HANDOFF_PREPARED=YES`; `BUSINESS_LEGAL_INPUTS_COMPLETE=NO`; `TESTED_SECURITY=NO`; `PRODUCTION_READY=NO`; `CODE_PERMISSION=NO`. Round 3 `BLOCKED` is historical (audited before these new decisions), not silently rewritten. A fresh final review must evaluate current sources, then user must explicitly authorize code.
