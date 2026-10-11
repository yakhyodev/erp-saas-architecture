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
