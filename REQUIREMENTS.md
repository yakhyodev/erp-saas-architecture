# REQUIREMENTS.md

- **Version:** 0.1 (2026-10-11)
- **Scope:** central cross-module registry; source main and user-provided 14-module audit
- **Governance:** APPROVED user business rules are binding; proposals are not approvals; missing provenance remains unresolved.
- **Source:** [Master Architecture](https://github.com/yakhyodev/erp-saas-architecture/blob/main/docs/modules/MASTER_ARCHITECTURE.md)

## APPROVED — functional and business requirements

| ID | Requirement | Status |
|---|---|---|
| BIZ-001 | Hybrid Prepaid SaaS monetizatsiyasi | APPROVED |
| BIZ-002 | Arzon Seat + Business Usage + Resource Usage | APPROVED |
| BIZ-003 | Business Usage hujjatga asosiy narx; hajm chegarasidan so‘ng qo‘shimcha kredit | APPROVED |
| BIZ-004 | 1 kredit = 10 UZS | APPROVED |
| BIZ-005 | Progressiv hajm chegirmalari va enterprise individual shartnomalar | APPROVED |
| BIZ-006 | Balans yetmaganda faqat haqiqiy Seat va Usage xarajatlari bir kalendar oy davomida minusga yoziladi | APPROVED |
| BIZ-007 | Minus balans uchun taxminiy kredit limiti hisoblanmaydi | APPROVED |
| BIZ-008 | Kompaniyalar va filiallar yagona SaaS platformasida | APPROVED |
| BIZ-009 | uz/en/ru interfeyslar va kelajakdagi yangi o‘zbek alifbosi uchun yashirin variant | APPROVED |
| BIZ-010 | Foydalanish qulayligi asosiy platforma maqsadi | APPROVED |

## PROPOSED — nonfunctional architecture requirements

| ID | Requirement (target values not approved) | Status |
|---|---|---|
| NFR-001 | Isolation: enforce tenant and company/branch authorization in backend APIs, jobs and reporting | PROPOSED |
| NFR-002 | Audit: trace billable facts and business posting/reversal with immutable references and idempotent processing | PROPOSED |
| NFR-003 | Performance: define per-workflow latency and throughput SLOs by benchmarking; no invented targets | PROPOSED |
| NFR-004 | Availability: define DB HA, backup, RPO/RTO after AR-003 | PROPOSED |
| NFR-005 | Usability: task-based acceptance for core workflows, localization, accessibility and error recovery | PROPOSED |
| NFR-006 | Scalability: measured multi-server evolution without premature microservice/sharding mandate | PROPOSED |
| NFR-007 | Offline: timestamping, event ordering, dedup and delayed-month pricing must align with Billing | PROPOSED |
| NFR-008 | External API: track evidence level separately from proposed behavior; do not claim unverified integration | PROPOSED |

## OPEN QUESTION — acceptance gates

- **AR-001:** Tenant / Company / BillingAccount / Wallet cardinality — owner: 02 Multi-Tenant + 01 Billing.
- **AR-002:** Catalog / Product ownership — owner: 05 Sales & Inventory + 07 CRM.
- **AR-003:** Initial database HA divergence — owner: 03 Database + 11 Infrastructure.
- **AR-004:** FIN-001 APPROVED provenance missing — owner: 06 Finance & Accounting + 00 Master.
- **AR-005:** Sales / Stock / GL lifecycle trigger matrix — owner: 05 Sales & Inventory + 06 Finance.
- **AR-006:** ADR-MOB-004 / ADR-MOB-009 missing — owner: 09 Mobile & Offline.
- **AR-007:** Missing root global registries — owner: 00 Master Architecture.
- **AR-008:** Party / Counterparty / Payer / Outlet master — owner: 07 CRM & Distribution + 06 Finance + 05 Sales.
- **AR-009:** Offline delayed usage calendar-month rating — owner: 09 Mobile & Offline + 01 Billing.
- **AR-010:** Chargeable document lifecycle — owner: 01 Billing + 05 Sales & Inventory.
- **AR-011:** External API evidence levels — owner: 12 Integrations & API + 10 Security.
- **AR-012:** UX workflow acceptance — owner: 08 UI/UX + 09 Mobile.
- **AR-013:** Identity / membership / permissions ownership — owner: 10 Security & Permissions + 02 Multi-Tenant.

Do not treat any numerical price, volume breakpoint, SLA, RPO/RTO, role definition, document charge trigger or storage partition as approved unless separately evidenced.


## Round 2 validation — v0.2 (2026-10-12)

- **APPROVED UX obligation:** Butun ERP platformasida foydalanuvchiga tushunarli va qulay tajriba (BIZ-010) majburiy mahsulot maqsadi. UX-T01…T20 matrix hujjatlashtirilgan, lekin user acceptance, pilot va metrikalar **OPEN** (MR-16/20); sifat o‘lchangani da'vo qilinmaydi.
- **AR-006:** ADR-MOB-004 va ADR-MOB-009 **EXISTS / FIXED_DOCUMENTATION**; offline TTL, device revocation, local wipe, retention va auth siyosati **PROPOSED/OPEN**, verified security emas.
- **AR-007:** beshta root registries GitHub main'da **EXISTS / FIXED_DOCUMENTATION**; sinxronizatsiya va provenance work davom etadi.
- **MR-18:** HR, Manufacturing, Purchasing, Reporting — har birining MVP inclusion/exclusion **OPEN QUESTION**; arxitektura xaritasida modul tilga olinishi MVP ga kiritish qarori emas.
- **Readiness:** Round 2 auditor verdict **BLOCKED**, 12 BLOCKED / 2 READY_WITH_CONDITIONS / 0 READY; yangi approval va bajarilgan test da'vosi yo'q. [Round 2 audit](docs/reviews/ARCHITECTURE_REVIEW_ROUND_2.md).


## v0.3 — Current-main module alignment and acceptance gaps (2026-10-12)

Current source: [current-main sync baseline](https://github.com/yakhyodev/erp-saas-architecture/commit/f26d4587dcad20f4bf7e8404e27cdcaeace9a355); older AR-006/AR-007 “missing” enumerations above are historical and superseded by the Round 2 v0.2 corrections. **APPROVED BIZ-001…010 remains unchanged.** Documented workflows and test plans are not acceptance results.

| Area | Documented PROPOSED requirement | Still OPEN / test gate |
|---|---|---|
| Tenant/Billing | Actual Seat + Business/Resource Usage only, no forecast credit cap; immutable source attribution and independent Billing ledger | Company/account/wallet, legally liable payer, anniversary calendar-month edge policy, seat daily proration, tier precedence |
| Domain data | Separate Catalog/Product/SKU, Pricing, Sales, Inventory, Purchasing, CRM/Party and Finance owner contracts | Canonical Catalog namespace, Party writer, GL/stock/accounting and Billing charge transitions |
| Event integrity | Owner-local transaction+outbox and scoped business_occurrence_id dedup preferred | Final approval, reconciliation test and lifecycle recognition |
| Identity & Security | Server authorization on all entrypoints; no frontend-only auth; tenant isolation and permission-safe search | Authoritative membership/grant writer, revoke freshness, RLS/pooling and negative test evidence |
| Mobile | Secure scoped cache, stable offline operation and server-side replay authorization | TTL, device policy, offline usage period, conflict and deletion policy |
| Integrations | Evidence categories DOCUMENTED/LIVE_VERIFIED/INFERRED/UNVERIFIED, source-specific SalesDoc V2 compatibility appendix | API/sandbox live verification, migration reconciliation, signed cutover |
| UX | Mandatory platform-wide usability, task-first workflows, uz/en/ru; hidden future alphabet remains OFF | Pilot and user-approved UX-T01…T20 priorities, a11y/device/network targets, execution evidence |
| Operations | Stage A recoverability ≠ HA; Stage B standby/fencing candidate, Stage C later | Initial stage, backup retention, region, RPO/RTO/SLO and measured load/DR tests |
| MVP | HR, Manufacturing, Purchasing and Reporting are candidate domains | Per-domain include/exclude, phase and ownership business decision |

Do not treat Billing §21's `SELECTED_PENDING_APPROVAL` choices as pricing or contract approvals. No implementation/release readiness asserted.

---

## v0.4 — MVP Company-scoped SaaS Billing requirements (2026-10-12)

**APPROVED_BY_USER:** [ADR-BILL-011](docs/adr/ADR-BILL-011.md) `BIZ-011` Company:LegalEntity:BillingAccount:Wallet 1:1:1:1 (Branch no Wallet); `BIZ-012` Company LegalEntity liable for SaaS fees despite authorized third-party Wallet top-up; `BIZ-013` per-user/per-Company Seat with branch/device dedup and independent different-Company Seats; `BIZ-014` Tenant-shared measured Resource Usage attributed to one predefined responsible Company and ordinary Company/Branch resources to their own Company. User approval source 2026-10-12 direct project conversation; [DECISIONS v0.4](DECISIONS.md). `MR-01` partially resolved cardinality/liability, `MR-03` scope resolved but billing status/proration/rates OPEN; `MR-04` categories/rates/measurement still OPEN. Proposed workflow acceptance: verified two-role responsibility acceptance, effective-dated immutable attribution, pending unresolved meter evidence, no fallback arbitrary Wallet or cross-company debt transfer. Runtime tests NOT EXECUTED.

---

## v0.5 — Seat lifecycle detailed candidate contract (MR-03; 2026-10-12)

**APPROVED_BY_USER (carried from BIZ-013/ADR-BILL-011):** Seat identity = canonical user × Company, no duplicate by Branch/device, separate Seat per Company. New [ADR-BILL-012](docs/adr/ADR-BILL-012.md) formalizes **PROPOSED**: effective membership transitions, activated-vs-pending classification, candidate daily proration, immutable day/period charge snapshots, per-economic-occurrence dedup, cross-company transfer and corrective reversal. **MR-03 remains PARTIALLY_RESOLVED** until status eligibility, suspension, cutoff timezone, same-day billing, monthly charge cadence, rate, rounding, refunds, enterprise tiers and full business approval are resolved. Invoice/negative grace period MR-02 is a separate open clock. Tests are design-only; no production code.

---

## v0.6 — delegated design baseline and Codex architecture handoff requirements (2026-10-12)

**APPROVED business:** existing user invariants plus explicit [ADR-BILL-011](docs/adr/ADR-BILL-011.md) MVP Company distinct LegalEntity/BillingAccount/Wallet, debt owner vs third-party funder, user×Company Seat no duplicate by branch/device and one designated responsible Company for Tenant-shared *actual measured* Resource Usage.

**DESIGN_SELECTED_UNDER_DELEGATION, official PROPOSED:** [ADR-010](docs/adr/ADR-010-mvp-delegated-baseline.md) Go modular monolith, Postgres transaction-scoped tenant guards/RLS, Next.js, Flutter, Party shared context, Catalog/Pricing independent, owner-local ACID+outbox, finance/stock/billing event boundaries, Stage A internal pilot PITR not HA, next Stage B when required, staged MVP scope; [BILL-013](docs/adr/ADR-BILL-013.md) anniversary clamp negative grace; [BILL-014](docs/adr/ADR-BILL-014.md) versioned final-document charge eligibility and marginal discounts; [BILL-012](docs/adr/ADR-BILL-012.md) Seat day calculation candidate. Until real prices are published, **no monetary debit** for candidate document/Seat/resource events. No arbitrary guessed Wallet or credit ceiling; no invoice second stock movement; no unapproved GL/tax posting.

[Codex handoff](docs/handoff/CODEX_START_HERE.md) is an architecture/spec package, **not application-code permission**. [MR-01…20 current gate matrix](docs/handoff/REMAINING_GATES.md) distinguishes APPROVED_USER, DESIGN_SELECTED, EXTERNAL_INPUT_REQUIRED and TEST_DEFINED_NOT_RUN. Legal contracting jurisdiction, taxes, company functional currency, fiscal GL profile, tariffs/tiers/refunds, RPO/RTO/region/backup retention, privacy/GPS retention, SalesDoc LIVE_VERIFIED and real security/performance/UX evidence remain mandatory appropriate-phase acceptance inputs. Historical Round 3 readiness BLOCKED is not retroactively modified.
