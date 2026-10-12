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


## v0.3 — Current-main AR-001…013 and MR-01…20 delta ledger (2026-10-12)

**Source:** [current-main sync baseline](https://github.com/yakhyodev/erp-saas-architecture/commit/f26d4587dcad20f4bf7e8404e27cdcaeace9a355); 01–13 Round 2 corrections. **Interpretation:** this current delta supersedes stale existence/conflict wording in historical tables above, not the audit document. `FIXED_DOCUMENTATION` ≠ user-approved; `SELECTED_PENDING_APPROVAL` ≠ APPROVED; `NOT_TESTED` ≠ failed or passed. No AR or MR business approval inferred.

| AR | Updated status | Owner | New evidence / required next action |
|---|---|---|---|
| AR-001 | HARMONIZED_PROPOSED / SELECTED_PENDING_APPROVAL | 01 Billing + 02 MT + 06 Finance | Company 1:1 BillingAccount 1:1 Wallet selected candidate; legal debtor/payer and shared liability OPEN |
| AR-002 | HARMONIZED_PROPOSED / OPEN | 04 Backend + 05 Sales + 07 CRM | Separate Catalog/Product/SKU and Pricing writers proposed; catalog namespace/container OPEN |
| AR-003 | HARMONIZED_PROPOSED / OPEN / NOT_TESTED | 03 DB + 11 Infra | Stage A single-primary PITR NOT HA; B standby/fencing; C later automation; A/B, RPO/RTO OPEN |
| AR-004 | MISSING_APPROVAL_PROVENANCE | 06 Finance + 00 Master | FIN-001 remains labelled APPROVED; original explicit approval evidence UNVERIFIED |
| AR-005 | HARMONIZED_PROPOSED / APPROVAL_OPEN | 03 DB + 04 Backend + 05 Sales + 06 Finance | DB §18 explicitly classifies §6 cross-domain ACID as historical non-selected PROPOSED; owner-local ACID+outbox preferred PROPOSED, GL trigger OPEN |
| AR-006 | FIXED_DOCUMENTATION / POLICY_OPEN | 09 Mobile + 10 Security | ADR-MOB-004/009 exist; offline TTL, revocation, device cleanup, conflict policy OPEN |
| AR-007 | FIXED_DOCUMENTATION / GOVERNANCE_SYNC | 00 Master + 14 Audit | Five registries exist; status/provenance reconciliation continues; not new approval |
| AR-008 | HARMONIZED_PROPOSED / OPEN | 07 CRM + 06 Finance + 12 Integrations | CRM Party writer vs shared Party context OPEN; Finance legal AR debtor, SaaS Billing payer distinct |
| AR-009 | SELECTED_PENDING_APPROVAL / OPEN | 01 Billing + 09 Mobile + 04 Backend | Server-approved business-effective month with linked current-open-period adjustment selected; closed-month, negative grace impact OPEN |
| AR-010 | HARMONIZED_PROPOSED / OPEN | 01 Billing + 05 Sales + 12 Integrations | Source lifecycle/occurrence and Billing charge separated; doc-type/transitions, refunds, partial/import OPEN |
| AR-011 | FIXED_DOCUMENTATION / LIVE_NOT_VERIFIED | 12 Integrations + 07 CRM | SalesDoc provider evidence appendix restored; DOCUMENTED ≠ LIVE_VERIFIED; cutover OPEN |
| AR-012 | FIXED_DOCUMENTATION / NOT_TESTED / OPEN | 08 UX + 09 Mobile + 13 Localization | UX-T01…T20 and a11y/search acceptance matrix documented, no measured pilot or approval |
| AR-013 | HARMONIZED_PROPOSED / OWNER_OPEN | 10 Security + 02 MT + 04 Backend | Security Identity/Auth + RoleAssignment/PolicyEvaluation, MT TenantMembership candidate; competing ownership not approved |

| MR | Current status | Owner | Decision type | Next action / cross-domain dependency |
|---|---|---|---|---|
| MR-01 | SELECTED_PENDING_APPROVAL | Billing+MT+Finance | BUSINESS | Company→BillingAccount→Wallet 1:1 candidate; debtor/payer mapping approval |
| MR-02 | SELECTED_PENDING_APPROVAL | Billing | BUSINESS | Anniversary calendar month; clamp/roll-forward, timezone, top-up, expiry OPEN |
| MR-03 | SELECTED_PENDING_APPROVAL | Billing+Security+MT | BUSINESS | Activated Seat + daily proration candidate; billing lifecycle OPEN |
| MR-04 | SPECIFIED_PROPOSED / OPEN | Billing+Sales+Integration | BUSINESS | Eligible document/resource list, unit/rate and transition OPEN |
| MR-05 | SELECTED_PENDING_APPROVAL | Billing | BUSINESS | Marginal progressive tiers candidate; tier window/stacking/contract precedence OPEN |
| MR-06 | SPECIFIED_PROPOSED / OPEN | Finance | BUSINESS/LEGAL | Tax jurisdiction, GL recognition, currency and close policy OPEN |
| MR-07 | HARMONIZED_PROPOSED / OPEN | DB+Backend+Sales+Finance | TECH + BUSINESS | Owner-local ACID/outbox favored, source/stock/GL triggers need sign-off |
| MR-08 | SPECIFIED_PROPOSED / OPEN | Sales+Inventory+Finance | BUSINESS | Reservation TTL, negative stock, valuation, count cutoff OPEN |
| MR-09 | SPECIFIED_PROPOSED / OPEN | CRM+Finance+Integration | OWNERSHIP/BUSINESS | Party writer/merge, ERP debtor, third-party payer OPEN |
| MR-10 | SPECIFIED_PROPOSED / UNTESTED | Backend+Integrations | TECHNICAL | Versioned event/API, occurrence/dedup and external compatibility testing |
| MR-11 | SPECIFIED_PROPOSED / NOT_TESTED | MT+DB+Security | TECH/SECURITY | Shared-schema/RLS/placement and negative tests OPEN |
| MR-12 | DOCUMENTATION_FIXED / POLICY_OPEN | Mobile+Security+Backend | SECURITY | Mobile ADRs exist; offline TTL/revoke/cleanup and sync conflicts OPEN |
| MR-13 | SPECIFIED_PROPOSED / OPEN | CRM+Mobile+Security | PRIVACY/BUSINESS | GPS lawful basis, geofence/retention/media/erasure OPEN |
| MR-14 | HARMONIZED_PROPOSED / NOT_TESTED | Infra+DB | SLA/BUSINESS | Stage A/B, RPO/RTO, budget, recovery/failover testing OPEN |
| MR-15 | SPECIFIED_PROPOSED / NOT_TESTED | Infra+DB | PERFORMANCE | Resource budgets, p95/p99 workload, concurrency and tests pending |
| MR-16 | DOCUMENTATION_FIXED / NOT_TESTED | UX+domain owners | UX/BUSINESS | UX-T01…T20 approved priority and pilot evidence OPEN |
| MR-17 | SPECIFIED_PROPOSED / NOT_TESTED | UX+Security+Localization | SECURITY/TECH | Search counts/facets/cache/revoke negative tests pending |
| MR-18 | OPEN_BUSINESS | Master+domain owners | MVP SCOPE | HR/Manufacturing/Purchasing/Reporting include/exclude OPEN |
| MR-19 | DOCUMENTATION_RESTORED / NOT_LIVE_VERIFIED | Integrations+Security | EXTERNAL EVIDENCE | SalesDoc V2 compatibility appendix documented, live testing/cutover OPEN |
| MR-20 | SPECIFIED_PROPOSED / NOT_TESTED | UX+Mobile+Localization | UX/BUSINESS | Accessibility/device/locale/network targets and pilot approval OPEN |

### User decision packet — priority proposal, NOT APPROVED

**P0**: (1) Company→BillingAccount→Wallet candidate and legal debtor/payer allocation; (2) anniversary one-calendar-month anchor, date clamp, timezone, refill/expiry and late usage month; (3) chargeable document type × lifecycle transition × reversals, resource units, tier precedence; (4) GL revenue/COGS/AR recognition, stock reservation/valuation and owner-local transaction approvals; (5) IdentityAuth/TenantMembership/RoleAssignment single-writer ownership, isolation + revoked-device policy; (6) Finance jurisdiction/tax/functional currency. **No estimated credit ceiling** under any option.

**P1**: Catalog global/tenant scope and Pricing container, Party owner/merge/legal payer mapping, Seat day proration details, discount tier window and enterprise precedence, HA Stage A/B and RPO/RTO/backup region/budget, Search permission leakage tests, SalesDoc documented-versus-live evidence and cutover, UX-T01…20 priority/a11y performance pilot, HR/Manufacturing/Purchasing/Reporting MVP scope, GPS privacy/offline retention. No numerical defaults are approved.

**Audit readiness:** Round 2 previously reported BLOCKED; this registry update is **not** a new independent Codex readiness review.

---

## v0.4 — Partial resolution of AR-001/MR-01/03/04 from direct user decisions (2026-10-12)

[ADR-BILL-011](docs/adr/ADR-BILL-011.md) documents **APPROVED_BY_USER** MVP Company→distinct LegalEntity→BillingAccount→Wallet 1:1:1:1 (no Branch Wallet), SaaS liability at the Company's LegalEntity, authorized third-party top-up without debt transfer, per-user/per-Company Seat (no Branch/device duplicates) and one designated responsible Company for Tenant-shared true Resource Usage.

- **AR-001 / MR-01 = PARTIALLY_RESOLVED:** MVP account cardinality and debt holder approved; precise legal identity verification, funder authorization/refund, enterprise consolidation and legal migration require separate sign-off.
- **MR-03 = PARTIALLY_RESOLVED:** per-Company seat count approved; active/suspend/revoke time, activated daily-prorating effective period, price, rounding still OPEN.
- **MR-04 = PARTIALLY_RESOLVED:** responsible-company source attribution approved; resource classes/units/metering window/rates, chargeable document eligibility/threshold/refunds remain OPEN.
- **Operational PROPOSED:** Tenant admin proposes; recipient Company's authorized finance representative accepts; mapping future effective, immutable historical liability; no automated cross-wallet transfer or arbitrary wallet fallback. Exact approval role delegation, next metering cutoff/timezone, pending attribution, closed-month correction, chargeback remain OPEN.

Earlier AR-001/MR-01 open wording is **historical for the particular now-approved scope**, not wholly deleted. All other AR/MR and FIN-001 provenance are untouched; Round 3 review snapshot still valid historically; tests not run.

---

## v0.5 — MR-03 Seat charging subdecisions and ADR-BILL-012 (2026-10-12)

Existing user-approved **Seat scope**: one user×Company, no branch/device duplication, independently billable across Companies ([ADR-BILL-011](docs/adr/ADR-BILL-011.md)). New [ADR-BILL-012](docs/adr/ADR-BILL-012.md) describes candidate ACTIVE start, invited/pending excluded, local calendar-day prorating, versioned membership events, canonical company-scoped charge dedup and append-only corrections. **No new commercial approval**; technical alternatives stay PROPOSED. **MR-03 = PARTIALLY_RESOLVED**: unresolved (1) activated day counted vs next day vs hourly (2) SUSPENDED reserved-paid vs paused (3) remove/transfer cutoff and same-day multiple company handling (4) local timezone/month window and billing cadence (5) daily fraction and rounding (6) exact Seat amount, enterprise discounts and any refunds (7) out-of-order/backdated correction and MR-02 negative grace interaction (8) final membership writer Security/MT sign-off AR-013. Do not claim Codex ready or executed tests.

---

## v0.6 — delegated design choices vs genuinely remaining external/evidence gates (2026-10-12)

**Review note:** Previous AR/MR `OPEN` descriptions are historical and may be **DESIGN_SELECTED_UNDER_DELEGATION**, but this is not the same as independently approved monetary/legal policy, technical owner sign-off, or executed tests. See [all 20 MR current status + closure owners](docs/handoff/REMAINING_GATES.md), [ADR-010](docs/adr/ADR-010-mvp-delegated-baseline.md), [BILL-013](docs/adr/ADR-BILL-013.md), [BILL-014](docs/adr/ADR-BILL-014.md).

- **AR-001/MR-01:** user-approved Company LegalEntity/BillingAccount/Wallet/Seat/sharing; payer authorization and legally compliant billing records E.
- **AR-002/MR-09:** Catalog dedicated and Party shared writer selected candidate; owner technical sign-off and legal counterparty checks S/E.
- **AR-003/MR-14/15:** internal Stage A non-HA and later B selected; business region/RPO/RTO/SLO and measured restore/performances E/T.
- **AR-004:** FIN-001 historical label has no independently evidenced original user approval; remains OPEN_PROVENANCE; do not rewrite past.
- **AR-005/MR-06/07/08:** source domain owner-local outbox selected, GL legal/accounting profile, stock valuation/negative/reversal and concurrency approvals E/S/T.
- **AR-006/009/MR-02/12/13:** offline revocation and latency rules selected; negative anniversary-clamp candidate, privacy/expiry/retention legal choices E/S, actual tests T.
- **AR-007:** new ADRs and current delta index created; historical files and old audit preserved; governance ongoing.
- **AR-010/MR-03/04/05:** per-user-per-Company Seat directly approved; activated-day candidate, document final-transition/marginal-tier defaults selected, but actual price/rate/threshold/tax/refund and rearm/expiry E.
- **AR-011/MR-19:** source-specific SalesDoc appendix DOCUMENTED only; production verification E/T.
- **AR-012/MR-16/17/20:** UX acceptance contract and launch task scope selected, user pilot/baseline/a11y tests T and real customer target E.
- **AR-013/MR-11:** Multi-Tenant membership, Security grants, Backend guard preferred; privileged DB/RLS/pool/caches/search cross-tenant negative tests and tech sign-off S/T.
- **MR-18:** MVP core Sales/Inventory/Purchasing/CRM/Billing/authorized core Finance, Mobile, UX + basic reporting selected; full HR/payroll/Manufacturing/advanced BI deferred by proposed roadmap.

**Remaining critical E/S/T gates:** contract fiscal jurisdiction, company legal/payer evidence, effective price books/discount tiers, provider settlement and refund rules, negative expiry terms, inventory valuation and GL tax point, placement/RLS authorization, backup restoration, quantitative SLO cost, mobile GPS lawful purpose and retention, SalesDoc live test and genuine UX pilot. No code writes were authorized by preparing [Codex start](docs/handoff/CODEX_START_HERE.md). **Do not mark AR/MR as TEST_PASSED or READY on a documentation basis**.
