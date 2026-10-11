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
