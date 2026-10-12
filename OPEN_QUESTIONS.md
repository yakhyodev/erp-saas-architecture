# OPEN_QUESTIONS.md

- **Version:** 0.1 (2026-10-11)
- **Scope:** central cross-module registry; source main and user-provided 14-module audit
- **Governance:** APPROVED user business rules are binding; proposals are not approvals; missing provenance remains unresolved.
- **Source:** [Master Architecture](https://github.com/yakhyodev/erp-saas-architecture/blob/main/docs/modules/MASTER_ARCHITECTURE.md)

## 14-module audit reconciliation — 13 unresolved items

- **Baseline supplied by audit:** `016573c8426ed3ea3b1eacbd0dc81adb67861220` (historical baseline, not assumed current HEAD).
- **All entries status:** OPEN QUESTION; no user approval inferred.

| ID | Issue | Responsible chat(s) | Status |
|---|---|---|---|
| AR-001 | Tenant / Company / BillingAccount / Wallet cardinality | 02 Multi-Tenant + 01 Billing | OPEN QUESTION |
| AR-002 | Catalog / Product ownership | 05 Sales & Inventory + 07 CRM | OPEN QUESTION |
| AR-003 | Initial database HA divergence | 03 Database + 11 Infrastructure | OPEN QUESTION |
| AR-004 | FIN-001 APPROVED provenance missing | 06 Finance & Accounting + 00 Master | OPEN QUESTION |
| AR-005 | Sales / Stock / GL lifecycle trigger matrix | 05 Sales & Inventory + 06 Finance | OPEN QUESTION |
| AR-006 | ADR-MOB-004 / ADR-MOB-009 missing | 09 Mobile & Offline | OPEN QUESTION |
| AR-007 | Missing root global registries | 00 Master Architecture | OPEN QUESTION |
| AR-008 | Party / Counterparty / Payer / Outlet master | 07 CRM & Distribution + 06 Finance + 05 Sales | OPEN QUESTION |
| AR-009 | Offline delayed usage calendar-month rating | 09 Mobile & Offline + 01 Billing | OPEN QUESTION |
| AR-010 | Chargeable document lifecycle | 01 Billing + 05 Sales & Inventory | OPEN QUESTION |
| AR-011 | External API evidence levels | 12 Integrations & API + 10 Security | OPEN QUESTION |
| AR-012 | UX workflow acceptance | 08 UI/UX + 09 Mobile | OPEN QUESTION |
| AR-013 | Identity / membership / permissions ownership | 10 Security & Permissions + 02 Multi-Tenant | OPEN QUESTION |

## Decision packets: variants, questions and dependencies

### AR-001 — Tenant / Company / BillingAccount / Wallet cardinality

- **Owner:** 02 Multi-Tenant + 01 Billing
- **PROPOSED options / review:** 1:N company per tenant; billing account tenantga yoki companyga; wallet 1:1 yoki 1:N. Entitlement/payment holder birligi tasdiqlansin.
- **OPEN QUESTION:** user/domain owner must decide; do not treat an option as approved.
- **Dependent domains:** Billing ownership, authorization, billing ledger.

### AR-002 — Catalog / Product ownership

- **Owner:** 05 Sales & Inventory + 07 CRM
- **PROPOSED options / review:** Global product catalog vs company-scoped catalog; stock SKU vs commercial product master; cross-company visibility.
- **OPEN QUESTION:** user/domain owner must decide; do not treat an option as approved.
- **Dependent domains:** Sales, Inventory, Purchase, CRM, integrations.

### AR-003 — Initial database HA divergence

- **Owner:** 03 Database + 11 Infrastructure
- **PROPOSED options / review:** Single-primary + backup/recovery vs initial HA primary/standby; RPO/RTO va o‘lchovlar.
- **OPEN QUESTION:** user/domain owner must decide; do not treat an option as approved.
- **Dependent domains:** DB, Infra, operations.

### AR-004 — FIN-001 APPROVED provenance missing

- **Owner:** 06 Finance & Accounting + 00 Master
- **PROPOSED options / review:** FIN-001 mavjud APPROVED holatini o‘zgartirmasdan asl explicit approval dalilini aniqlash; tasdiq bo‘lmasa statusni alohida ko‘rib chiqish.
- **OPEN QUESTION:** user/domain owner must decide; do not treat an option as approved.
- **Dependent domains:** Finance, Billing, decision governance.

### AR-005 — Sales / Stock / GL lifecycle trigger matrix

- **Owner:** 05 Sales & Inventory + 06 Finance
- **PROPOSED options / review:** Draft, confirm, ship, receive, invoice, return, cancel, reverse bosqichlarida stock/GL effect aniq jadvali.
- **OPEN QUESTION:** user/domain owner must decide; do not treat an option as approved.
- **Dependent domains:** Sales, Inventory, Finance, Purchase.

### AR-006 — ADR-MOB-004 / ADR-MOB-009 missing

- **Owner:** 09 Mobile & Offline
- **PROPOSED options / review:** ADRlar mavjudligi va oldingi havolalar tekshirilsin; hujjat bo‘lmasa mobil modul egasi yozsin.
- **OPEN QUESTION:** user/domain owner must decide; do not treat an option as approved.
- **Dependent domains:** Mobile, sync, API.

### AR-007 — Missing root global registries

- **Owner:** 00 Master Architecture
- **PROPOSED options / review:** REQUIREMENTS, DECISIONS, MODULE_REGISTRY, OPEN_QUESTIONS, CHANGELOG baseline yaratish; keyingi auditda cross-module sync.
- **OPEN QUESTION:** user/domain owner must decide; do not treat an option as approved.
- **Dependent domains:** Architecture governance, all modules.

### AR-008 — Party / Counterparty / Payer / Outlet master

- **Owner:** 07 CRM & Distribution + 06 Finance + 05 Sales
- **PROPOSED options / review:** Party registry yagona yoki bounded-context identity; payer, counterparty va outlet rol/entity mapping.
- **OPEN QUESTION:** user/domain owner must decide; do not treat an option as approved.
- **Dependent domains:** CRM, Sales, Finance, Billing, integrations.

### AR-009 — Offline delayed usage calendar-month rating

- **Owner:** 09 Mobile & Offline + 01 Billing
- **PROPOSED options / review:** Event occurred_at vs received_at va rating month; timezone, replay, reversal, late sync policy.
- **OPEN QUESTION:** user/domain owner must decide; do not treat an option as approved.
- **Dependent domains:** Mobile, Billing, reporting, audit.

### AR-010 — Chargeable document lifecycle

- **Owner:** 01 Billing + 05 Sales & Inventory
- **PROPOSED options / review:** Charge point: created, confirmed, posted yoki finalized; amendments, returns va duplicate commands.
- **OPEN QUESTION:** user/domain owner must decide; do not treat an option as approved.
- **Dependent domains:** Billing, Sales, Inventory, Finance.

### AR-011 — External API evidence levels

- **Owner:** 12 Integrations & API + 10 Security
- **PROPOSED options / review:** Contract schema vs sandbox-tested vs production-verified levels; claim/implementation gate.
- **OPEN QUESTION:** user/domain owner must decide; do not treat an option as approved.
- **Dependent domains:** Integrations, security, deployment.

### AR-012 — UX workflow acceptance

- **Owner:** 08 UI/UX + 09 Mobile
- **PROPOSED options / review:** Task-based acceptance, accessibility, error/recovery flows, language acceptance criteria.
- **OPEN QUESTION:** user/domain owner must decide; do not treat an option as approved.
- **Dependent domains:** UI/UX, mobile, security, domains.

### AR-013 — Identity / membership / permissions ownership

- **Owner:** 10 Security & Permissions + 02 Multi-Tenant
- **PROPOSED options / review:** Identity global vs per-tenant, membership owner, role+permission scope, branch context switching.
- **OPEN QUESTION:** user/domain owner must decide; do not treat an option as approved.
- **Dependent domains:** Tenant, Security, all modules.


## Approval process

1. Owner locates current local ADR / contract and describes differences and migration risks.
2. Master checks against approved business rules, provenance and impacted module contracts.
3. User explicitly approves any business/ownership/price/limit/charge trigger change; technical sign-off goes through corresponding ADR process.
4. Update owning ADR first, then DECISIONS, MODULE_REGISTRY, REQUIREMENTS, Master and CHANGELOG as relevant.
5. Verify written files on main. Do not rewrite historical APPROVED entries or silently close issues.


## Round 2 corrections — AR-006 / AR-007 (v0.2)

- **AR-006 = FIXED_DOCUMENTATION (physical files), OPEN_SECURITY_POLICY:** [ADR-MOB-004](docs/adr/ADR-MOB-004.md), [ADR-MOB-009](docs/adr/ADR-MOB-009.md) both exist on `main` and are **PROPOSED**. Old row above saying `missing ADRs` is **historical and superseded as file-existence claim**. Offline TTL, revoke/cleanup, device trust, retention and sync conflict authorization remain OPEN; owner 09 Mobile + 10 Security + 04 Backend.
- **AR-007 = FIXED_DOCUMENTATION (physical registries), OPEN_GOVERNANCE_SYNC:** `REQUIREMENTS.md`, `DECISIONS.md`, `MODULE_REGISTRY.md`, `OPEN_QUESTIONS.md`, `CHANGELOG.md` exist on `main`. Historical `missing registries` row above is superseded as existence claim. Status/provenance indexing and module syncing require ongoing review, not new APPROVED status.
- **Round 2 base:** [Independent audit](docs/reviews/ARCHITECTURE_REVIEW_ROUND_2.md) covers 97 ADR / 17 module docs; no production evidence was collected. Current readiness remains **BLOCKED**. Other AR-001..005/008..013 remain unresolved.

## MR-01…MR-20 decision and evidence registry (all not approved)

Status vocabulary follows the Round 2 audit: `SPECIFIED_PROPOSED` = contract drafted, `OPEN_BUSINESS` / `OPEN_TECH` = pending decisions, `NOT_TESTED` = no test evidence, `DOCUMENTATION_FIXED` = document exists only. **Evidence** column points to design documents, not runtime verification. Owner/dependency/action mappings below are coordination proposals.

| ID | Requirement | Audit status | Owner chat | Dependency | Decision type | Evidence (Round 2 §7) | Next action |
|---|---|---|---|---|---|---|---|
| MR-01 | Tenant/Company/LegalEntity/BillingAccount | SPECIFIED_PROPOSED + OPEN_BUSINESS | 01 Billing + 02 Multi-Tenant + 06 Finance | Tenant/Company/LegalEntity/BillingAccount/Wallet | BUSINESS + DATA OWNERSHIP | Round 2 §7: [MT §15](../modules/MULTI_TENANT_ARCHITECTURE.md), [Billing §19.1](../modules/BILLING_ARCHITECTURE.md) A/B/C/D mapping variantlari | Approve payer/account/wallet cardinality |
| MR-02 | Negative calendar month lifecycle | SPECIFIED_PROPOSED + OPEN_BUSINESS | 01 Billing | Billing month, timezone, balance | BUSINESS | Round 2 §7: [Billing §19.4](../modules/BILLING_ARCHITECTURE.md), [ADR-BILL-006](../adr/ADR-BILL-006.md) T0–T5, named calendar/anniversary variants | Set month anchor, expiry and timezone |
| MR-03 | Seat statuses/proration | SPECIFIED_PROPOSED + OPEN_BUSINESS | 01 Billing | Membership, Billing seat, tenant | BUSINESS | Round 2 §7: [Billing §19.5](../modules/BILLING_ARCHITECTURE.md), [ADR-BILL-010](../adr/ADR-BILL-010.md) activate/suspend/seat dedup | Decide billable seat states/proration |
| MR-04 | Document & Resource Usage classification | SPECIFIED_PROPOSED + OPEN_BUSINESS | 01 Billing + 05 Sales + 12 Integrations | Sales docs, resource meter, import | BUSINESS | Round 2 §7: [Billing §19.2/§19.6](../modules/BILLING_ARCHITECTURE.md), [ADR-BILL-009](../adr/ADR-BILL-009.md) event matrix | Choose billable events/resource types |
| MR-05 | Progressive discounts/enterprise | SPECIFIED_PROPOSED + OPEN_BUSINESS | 01 Billing | Billing pricing, enterprise terms | BUSINESS | Round 2 §7: [Billing §19.6](../modules/BILLING_ARCHITECTURE.md), [ADR-BILL-007](../adr/ADR-BILL-007.md) marginal vs retroactive | Approve discounts and stacking |
| MR-06 | Finance jurisdiction, legal GL | SPECIFIED_PROPOSED + OPEN_BUSINESS | 06 Finance | Sales/Stock/GL, tax | BUSINESS / LEGAL | Round 2 §7: [FINANCE_ARCHITECTURE addendum](../modules/FINANCE_ARCHITECTURE.md), [FIN-002](../adr/FIN-002.md) AR/GL source mapping | Approve jurisdiction, GL policies |
| MR-07 | Sales/Stock/Finance effects | SPECIFIED_PROPOSED + OPEN_TECH + OPEN_BUSINESS | 03 DB + 04 Backend + 05 Sales + 06 Finance | Sales/Inventory/Finance/Database | BUSINESS + TECHNICAL | Round 2 §7: [Sales §11.2–11.3](../modules/SALES_INVENTORY.md), [Database §6/§17](../modules/DATABASE_ARCHITECTURE.md), [ADR-SI-012](../adr/ADR-SI-012.md) | Reconcile DB §6 vs §17 and GL trigger |
| MR-08 | Stock reservation/valuation/count | SPECIFIED_PROPOSED + OPEN_BUSINESS | 05 Sales & Inventory | Stock, Finance valuation | BUSINESS | Round 2 §7: [Sales §11.5](../modules/SALES_INVENTORY.md), [ADR-SI-004](../adr/ADR-SI-004.md), [ADR-SI-008](../adr/ADR-SI-008.md) | Approve negative stock, TTL, valuation |
| MR-09 | Party/customer/payer master | SPECIFIED_PROPOSED + OPEN_BUSINESS | 07 CRM + 06 Finance + 12 Integrations | CRM/Finance/Integration/Party | BUSINESS + OWNERSHIP | Round 2 §7: [CRM §11](../modules/CRM_DISTRIBUTION.md), [Finance Accounting](../modules/FINANCE_ACCOUNTING.md) | Approve canonical Party/payer |
| MR-10 | API/event compatibility | SPECIFIED_PROPOSED + OPEN_TECH | 04 Backend + 12 Integrations | Backend API, integrations, event delivery | TECHNICAL | Round 2 §7: [Backend §17](../modules/BACKEND_ARCHITECTURE.md), [Integration — MR-10](../modules/INTEGRATION_ARCHITECTURE.md) | Approve versioned API/event catalog |
| MR-11 | Isolation/RLS/placement | SPECIFIED_PROPOSED + OPEN_TECH + NOT_TESTED | 02 Multi-Tenant + 03 DB + 10 Security | Tenant, DB pooling, Security | TECHNICAL + TEST EVIDENCE | Round 2 §7: [Security §19](../modules/SECURITY_ARCHITECTURE.md), [MT ADR-001](../adr/ADR-MT-001.md), [DB §17](../modules/DATABASE_ARCHITECTURE.md) | Select isolation and conduct RLS tests |
| MR-12 | Mobile offline sync/session | DOCUMENTATION_FIXED + SPECIFIED_PROPOSED + OPEN_TECH | 09 Mobile + 10 Security + 04 Backend | Mobile sync, Security revocation | TECHNICAL + SECURITY | Round 2 §7: [ADR-MOB-004](../adr/ADR-MOB-004.md), [ADR-MOB-009](../adr/ADR-MOB-009.md) | Approve TTL, revocation and sync policy |
| MR-13 | GPS/media/privacy | SPECIFIED_PROPOSED + OPEN_BUSINESS | 07 CRM + 09 Mobile + 10 Security | Mobile, CRM, legal privacy | BUSINESS / PRIVACY | Round 2 §7: [CRM §4/§6](../modules/CRM_DISTRIBUTION.md), [ADR-CRM-002](../adr/ADR-CRM-002-offline-gps-privacy.md), [ADR-MOB-006](../adr/ADR-MOB-006.md) | Decide lawful basis and retention |
| MR-14 | HA/SLO/RPO/RTO | SPECIFIED_PROPOSED + OPEN_BUSINESS + NOT_TESTED | 03 DB + 11 Infrastructure | Infra, DB, SLA | BUSINESS SLA + TECHNICAL | Round 2 §7: [Infra §18](../modules/INFRA_ARCHITECTURE.md), [ADR-DB-007](../adr/ADR-DB-007.md) | Select Stage A/B, RPO/RTO and DR drills |
| MR-15 | Peak load/performance | SPECIFIED_PROPOSED + OPEN_TECH + NOT_TESTED | 03 DB + 11 Infrastructure | Infra, DB, UX load | TECHNICAL + PERFORMANCE | Round 2 §7: [Infra §6/§12/§18](../modules/INFRA_ARCHITECTURE.md), [DB §7](../modules/DATABASE_ARCHITECTURE.md) | Set workload SLO, benchmark |
| MR-16 | Top-20 UX tasks | DOCUMENTATION_FIXED + SPECIFIED_PROPOSED + OPEN_BUSINESS + NOT_TESTED | 08 UI/UX + business owners | UX, business workflows, mobile | BUSINESS UX + EVIDENCE | Round 2 §7: [UX §16.1](../modules/UX_ARCHITECTURE.md) T01–T20 | Approve T01–T20 priority and pilot |
| MR-17 | Global search security | SPECIFIED_PROPOSED + OPEN_TECH + NOT_TESTED | 08 UI/UX + 10 Security + 13 Localization | Search, localization, Security | TECHNICAL + SECURITY | Round 2 §7: [UX §16.3](../modules/UX_ARCHITECTURE.md), [Localization §15](../modules/LOCALIZATION_ARCHITECTURE.md), [ADR-UX-003](../adr/ADR-UX-003.md) | Choose search index ownership, security tests |
| MR-18 | HR/Manufacturing/Reporting/Purchasing scope | OPEN_BUSINESS | 00 Master + 04 Backend + business owners | MVP roadmap, Purchasing/Reporting/HR/Manufacturing | BUSINESS SCOPE | Round 2 §7: [Master §2/§12](../modules/MASTER_ARCHITECTURE.md), [Backend §3](../modules/BACKEND_ARCHITECTURE.md) | Approve MVP inclusion/exclusion |
| MR-19 | SalesDoc evidence/cutover | SPECIFIED_PROPOSED + OPEN_TECH + NOT_TESTED | 12 Integrations + 10 Security | External API, evidence, cutover | TECHNICAL + EXTERNAL EVIDENCE | Round 2 §7: [Integration taxonomy + cutover](../modules/INTEGRATION_ARCHITECTURE.md), [ADR-INT-003](../adr/ADR-INT-003.md) | LIVE_VERIFY SalesDoc, authorize cutover |
| MR-20 | A11y/device/network/locale baseline | SPECIFIED_PROPOSED + OPEN_BUSINESS + NOT_TESTED | 08 UI/UX + 09 Mobile + 13 Localization | UX, Mobile, Localization | BUSINESS UX + EVIDENCE | Round 2 §7: [UX §16.5–16.6](../modules/UX_ARCHITECTURE.md), [Localization §15](../modules/LOCALIZATION_ARCHITECTURE.md) | Approve accessibility/device/network/locale gates |

### Priority review queue (P0/P1 proposed; not user priority approval)

- **P0:** AR-001/MR-01 payer/wallet cardinality; AR-005/MR-07 stock+GL lifecycle and transaction boundary; AR-009/MR-02 late usage/calendar month; AR-010/MR-04 chargeable event trigger; AR-013/MR-11 identity membership role ownership/isolation; MR-06 Finance jurisdiction and legal GL.
- **P1:** AR-002 Catalog/Product/Pricing; AR-008 Party/Payer; MR-03 seat proration; MR-05 progressive discounts; MR-14 DR objectives; MR-18 MVP scope; MR-16/20 UX quality gates; MR-12/13 offline/device/privacy; MR-19 external verification. Priorities are **PROPOSED** coordination labels, not approved sequencing.

### MVP inclusion/exclusion decisions — OPEN

| Domain | Include in MVP? | Exclude from MVP? | Final status |
|---|---|---|---|
| HR | Not decided | Not decided | OPEN QUESTION |
| Manufacturing | Not decided | Not decided | OPEN QUESTION |
| Purchasing | Not decided | Not decided | OPEN QUESTION |
| Reporting | Not decided | Not decided | OPEN QUESTION |
