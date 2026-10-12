# MVP Implementation Backlog — staged, testable, dependency-ordered

- **Status:** ARCHITECTURE HANDOFF PLAN ONLY, no product code authorized
- **Owner:** 00 Master, coordinated with 01–13
- **Reference:** [Codex start](CODEX_START_HERE.md), [Delegated MVP baseline ADR-010](../adr/ADR-010-mvp-delegated-baseline.md), [Remaining gates](REMAINING_GATES.md)

## Phase 0 — governance, runtime and business contract preflight (before application code)
- Freeze real user APPROVED invariants separately from `DESIGN_SELECTED_UNDER_DELEGATION` drafts. Resolve tech ADR ownership/tenant isolation acceptance; provide a signed contract for monetized/billable state, and fiscal/legal profile for enabled finance functions.
- Define target app repository, delivery environments, provider/secrets and jurisdiction only with real authority; never create false legal/commercial data. Establish risk-owned test plan for auth, tenant/branch, outbox, stock, wallet, accounting, UI and restore.
- **Gate output:** traceability ledger ADR→owner→API→test→effective policy and staged feature flags; no money path without published price policy.

## Stage 1 — core platform skeleton (after explicit separate code authorization)
Owner: Backend, Database, Tenant, Security. Go modular monolith boundaries, PostgreSQL migration + composite tenant keys, source-domain transaction and outbox/inbox contract; simple Web Next.js shell and Flutter shell; locale bundles `uz/en/ru` and error code catalog; local dev and CI security checks.
- Test contracts: scoped auth, tenant A/B branch leakage negative matrix; pooled DB transaction context/rollback; optimistic version conflicts; API event/command version evolution.
- Feature flags: `billing_posting=false`, `finance_gl_posting=false`, `external_writes=false`, `offline_mutation_sync=false` until gates.

## Stage 2 — business identity, Catalog and CRM
Owner: shared Party master, Catalog, CRM, Pricing, Tenant.
- Canonical Party with tenant-scoped identity, Customer/Outlet relationships and external mapping; canonical Product/SKU/UoM, tenant namespace and PriceList with immutable quote price snapshots; Company/Branch legal mapping with Finance verification.
- Test: no dual writer; duplicate merge maintains financial identity snapshots; branch-limited autocomplete/count, locale aliases never alter identifiers; cross-company Company and LegalEntity validate.
- Gate: Customer/Outlet != legal debtor; payer != funder != SaaS debtor.

## Stage 3 — Sales, Inventory and Purchasing event integrity
Owner: Sales, Inventory, Purchasing, Backend, Database.
- Sales draft→confirm with commercial state only; optional reservation; inventory dispatch only one physical movement per dispatch leg; goods receipt increment once; transfer physical legs; returns reverse linked original.
- Stock reservations prevent oversubscription under concurrency; negative stock disabled by selected policy until allowed; valuation moving-average candidate only after accounting sign-off. Purchasing minimal accepted goods receipts.
- Test: concurrent hot SKU, partial delivery, reversal, duplicate/out-of-order events, worker restart, no invoice second stock decrement; create detailed reconciliation report.

## Stage 4 — SaaS Wallet and exact economic effects
Owner: Billing, MT, Finance legal link, Security, Payment integration.
- Company→LegalEntity→BillingAccount→Wallet, unique user×Company Seat, versioned candidate monthly/day accrual, authorized third-party settled topup; Tenant-shared responsible Company two-step selected approval; actual usage meter fact and policy selection. Negative grace ADR-BILL-013 and document/tier ADR-BILL-014 only after published rates and formal economic go/no-go.
- Test: from zero negative, 28/29/30/31, legal timezone, repeated top-up, idempotent double delivery, wallet isolation, chargeable docs partial/import/return, applied enterprise discount, no unpriced charge, no credit ceiling.

## Stage 5 — ERP Finance + accounting compliance
Owner: Finance, Sales, Inventory, Purchasing, legal/accounting advisor.
- Append-only balanced GL postings, AR/AP, legally compliant invoice/credit note/recognition tax point, fiscal period close and separate SaaS Wallet ledger. **Disabled for real posting until jurisdiction, legal entity books, functional currency, tax profile, moving average/FIFO policy, chart-of-accounts and accounting owner sign-off**.
- Test: invoice only once, COGS vs revenue recognition, close/reopen, credit note corrections, debtor snapshot after Party merge, reconciliation vs original physical fact, delayed posting and duplicate source.

## Stage 6 — UX/Mobile/Offline/Localization
Owner: UX, Mobile, Security, Localization, domain owners.
- Task-first roles, actionable dashboards, per-Company wallet and branch attribution, status distinctions `QUEUED/UNKNOWN/REJECTED/ACCEPTED/POSTED`; optimistic but never deceptive settlement or stock view. Mobile encrypted offline outbox with stable operation ID, reauthorization on reconnect, no local GL/wallet effects.
- Test: UX-T01…UX-T20 by persona/role/device/locale/network; revoked device denied queued mutations; old branch cache cannot bleed into new; screen reader/keyboard focus and long Russian/Uzb texts; real paired Sales Doctor usability baseline needed before external superiority claim.

## Stage 7 — reporting, SalesDoc adapter, deployment readiness
Owner: Infrastructure, Integrations, Security, Product QA.
- Tenant authorized read-model reporting and global search (including counts/facets/recent); SalesDoc documented-only read-first adapter verified in authorized sandbox before any write/cutover; no uncontrolled data migration.
- Infrastructure: stage A verified offsite restore for pilot, selected Stage B HA before contractual uptime required; measured pool, queue, stock, billing month-end, hot tenant, mobile sync load. Use dedicated enterprise isolation only with placement/restore evidence.
- Test: DR restore drill, RPO/RTO evidence, failover fencing when B selected; search tenant leakage and no stale resource access; billing/GL/stock reconciliation.

## Vertical release gates

Each stage needs: (a) documented owner and immutable economic occurrence (b) versioned typed contract (c) auth/tenant negative cases (d) at-least-once retry/dedup test (e) UX and recovery-state acceptance (f) affordability/performance data (g) domain + legal approval for business-effective money/stock/tax path. An unverified feature stays disabled, never auto-enables based on elapsed time.

## MVP inclusion/exclusion (design-selected, not contractual commitment)

**Included candidate:** Tenant/Auth, core Catalog/Pricing, CRM Party/Outlet, Sales, Inventory, minimal Purchasing/receipt, SaaS Billing/Seat/Wallet, basic ERP Finance behind legal gate, UX, mobile sync and actionable dashboards. **Deferred candidate:** Manufacturing, payroll/full HR, sophisticated BI, production two-way SalesDoc sync, shared/pooled enterprise Wallet, dedicated DB as universal default. Keep clean extension points, no empty pretend-complete modules.

## Codex PR size & definition of done

Build one bounded context/use-case per PR, not a giant ERP generator; include docs, migrations only when authorized, rollback, unit/integration tests and observability. Never mark `READY` based on prose alone; tests must be actually executed in environment and results captured. Every money path should have golden-case and hostile-case deterministic fixtures with fixed precision arithmetic and no float money.
