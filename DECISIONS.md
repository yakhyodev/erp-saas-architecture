# DECISIONS.md

- **Version:** 0.1 (2026-10-11)
- **Scope:** central cross-module registry; source main and user-provided 14-module audit
- **Governance:** APPROVED user business rules are binding; proposals are not approvals; missing provenance remains unresolved.
- **Source:** [Master Architecture](https://github.com/yakhyodev/erp-saas-architecture/blob/main/docs/modules/MASTER_ARCHITECTURE.md)

## APPROVED — direct user business decisions

| ID | Decision | Status | Provenance |
|---|---|---|---|
| BIZ-001 | Hybrid Prepaid SaaS monetizatsiyasi | APPROVED | User project instructions / explicit user statements |
| BIZ-002 | Arzon Seat + Business Usage + Resource Usage | APPROVED | User project instructions / explicit user statements |
| BIZ-003 | Business Usage hujjatga asosiy narx; hajm chegarasidan so‘ng qo‘shimcha kredit | APPROVED | User project instructions / explicit user statements |
| BIZ-004 | 1 kredit = 10 UZS | APPROVED | User project instructions / explicit user statements |
| BIZ-005 | Progressiv hajm chegirmalari va enterprise individual shartnomalar | APPROVED | User project instructions / explicit user statements |
| BIZ-006 | Balans yetmaganda faqat haqiqiy Seat va Usage xarajatlari bir kalendar oy davomida minusga yoziladi | APPROVED | User project instructions / explicit user statements |
| BIZ-007 | Minus balans uchun taxminiy kredit limiti hisoblanmaydi | APPROVED | User project instructions / explicit user statements |
| BIZ-008 | Kompaniyalar va filiallar yagona SaaS platformasida | APPROVED | User project instructions / explicit user statements |
| BIZ-009 | uz/en/ru interfeyslar va kelajakdagi yangi o‘zbek alifbosi uchun yashirin variant | APPROVED | User project instructions / explicit user statements |
| BIZ-010 | Foydalanish qulayligi asosiy platforma maqsadi | APPROVED | User project instructions / explicit user statements |

## PROPOSED — technology and overarching ADRs

| ID | Decision | Status | Details |
|---|---|---|---|
| TECH-001 | Initial modular monolith; clearly-owned domains | PROPOSED | [Master](https://github.com/yakhyodev/erp-saas-architecture/blob/main/docs/modules/MASTER_ARCHITECTURE.md) |
| TECH-002 | Go backend | PROPOSED | [Master](https://github.com/yakhyodev/erp-saas-architecture/blob/main/docs/modules/MASTER_ARCHITECTURE.md) |
| TECH-003 | PostgreSQL transactional store | PROPOSED | [Master](https://github.com/yakhyodev/erp-saas-architecture/blob/main/docs/modules/MASTER_ARCHITECTURE.md) |
| TECH-004 | Next.js web | PROPOSED | [Master](https://github.com/yakhyodev/erp-saas-architecture/blob/main/docs/modules/MASTER_ARCHITECTURE.md) |
| TECH-005 | Flutter mobile | PROPOSED | [Master](https://github.com/yakhyodev/erp-saas-architecture/blob/main/docs/modules/MASTER_ARCHITECTURE.md) |
| TECH-006 | Tenant-aware isolation model; dedicated enterprise path | PROPOSED | Exact cardinality, DB boundary unresolved |
| TECH-007 | Measured multiserver and selective microservice extraction | PROPOSED | No fixed extraction date |
| TECH-008 | Billing module owns SaaS usage rating, not ERP GL | PROPOSED | Boundaries subject to AR-001/009/010 |

## Historical status / provenance exception

- **FIN-001** — audit reports `APPROVED` in the Finance ADR, but independent explicit approval provenance has not been established (**AR-004 OPEN QUESTION**). **Do not silently downgrade, upgrade, or rewrite the original ADR.** This registry preserves the reported status as `APPROVED (provenance unverified)`, pending source-of-truth review; this is *not* a new approval.
- Other module ADR statuses require individual source verification. The 14-audit count (108 Markdown files, 90 ADRs) is an audit snapshot, **not** a fresh exhaustive inventory confirmation.

## OPEN QUESTION register

- **AR-001** — Tenant / Company / BillingAccount / Wallet cardinality; owner: 02 Multi-Tenant + 01 Billing; see [OPEN_QUESTIONS.md](https://github.com/yakhyodev/erp-saas-architecture/blob/main/OPEN_QUESTIONS.md).
- **AR-002** — Catalog / Product ownership; owner: 05 Sales & Inventory + 07 CRM; see [OPEN_QUESTIONS.md](https://github.com/yakhyodev/erp-saas-architecture/blob/main/OPEN_QUESTIONS.md).
- **AR-003** — Initial database HA divergence; owner: 03 Database + 11 Infrastructure; see [OPEN_QUESTIONS.md](https://github.com/yakhyodev/erp-saas-architecture/blob/main/OPEN_QUESTIONS.md).
- **AR-004** — FIN-001 APPROVED provenance missing; owner: 06 Finance & Accounting + 00 Master; see [OPEN_QUESTIONS.md](https://github.com/yakhyodev/erp-saas-architecture/blob/main/OPEN_QUESTIONS.md).
- **AR-005** — Sales / Stock / GL lifecycle trigger matrix; owner: 05 Sales & Inventory + 06 Finance; see [OPEN_QUESTIONS.md](https://github.com/yakhyodev/erp-saas-architecture/blob/main/OPEN_QUESTIONS.md).
- **AR-006** — ADR-MOB-004 / ADR-MOB-009 missing; owner: 09 Mobile & Offline; see [OPEN_QUESTIONS.md](https://github.com/yakhyodev/erp-saas-architecture/blob/main/OPEN_QUESTIONS.md).
- **AR-007** — Missing root global registries; owner: 00 Master Architecture; see [OPEN_QUESTIONS.md](https://github.com/yakhyodev/erp-saas-architecture/blob/main/OPEN_QUESTIONS.md).
- **AR-008** — Party / Counterparty / Payer / Outlet master; owner: 07 CRM & Distribution + 06 Finance + 05 Sales; see [OPEN_QUESTIONS.md](https://github.com/yakhyodev/erp-saas-architecture/blob/main/OPEN_QUESTIONS.md).
- **AR-009** — Offline delayed usage calendar-month rating; owner: 09 Mobile & Offline + 01 Billing; see [OPEN_QUESTIONS.md](https://github.com/yakhyodev/erp-saas-architecture/blob/main/OPEN_QUESTIONS.md).
- **AR-010** — Chargeable document lifecycle; owner: 01 Billing + 05 Sales & Inventory; see [OPEN_QUESTIONS.md](https://github.com/yakhyodev/erp-saas-architecture/blob/main/OPEN_QUESTIONS.md).
- **AR-011** — External API evidence levels; owner: 12 Integrations & API + 10 Security; see [OPEN_QUESTIONS.md](https://github.com/yakhyodev/erp-saas-architecture/blob/main/OPEN_QUESTIONS.md).
- **AR-012** — UX workflow acceptance; owner: 08 UI/UX + 09 Mobile; see [OPEN_QUESTIONS.md](https://github.com/yakhyodev/erp-saas-architecture/blob/main/OPEN_QUESTIONS.md).
- **AR-013** — Identity / membership / permissions ownership; owner: 10 Security & Permissions + 02 Multi-Tenant; see [OPEN_QUESTIONS.md](https://github.com/yakhyodev/erp-saas-architecture/blob/main/OPEN_QUESTIONS.md).

Status transitions require user approval, record of evidence, superseding ADR (if changing an approved contract) and coordinated registry updates.


## Round 2 — full ADR status / approval provenance index (v0.2)

**Evidence basis:** [Round 2 audit §4](docs/reviews/ARCHITECTURE_REVIEW_ROUND_2.md), audited source commit `184b32f7b3dcc9ce8d329ca2dfb7895f06cae905` (97 ADR snapshot), not live implementation. This is one-row-per-ADR **status/provenance index**, not duplicate ADR summaries. Exact artifact filenames (including CRM slugged ones) belong to owner module and linked audit inventory; IDs are canonical here. 96 PROPOSED, one FIN-001 tagged APPROVED without independently verified explicit user approval.

| ADR ID | Owner | Document status | Provenance evidence | Next governance action |
|---|---|---|---|---|
| ADR-001 | Master | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-002 | Master | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-003 | Master | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-004 | Master | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-005 | Master | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-006 | Master | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-007 | Master | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-008 | Master | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-009 | Master | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-BILL-001 | Billing | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-BILL-002 | Billing | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-BILL-003 | Billing | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-BILL-004 | Billing | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-BILL-005 | Billing | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-BILL-006 | Billing | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-BILL-007 | Billing | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-BILL-008 | Billing | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-BILL-009 | Billing | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-BILL-010 | Billing | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-MT-001 | Multi-Tenant | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-MT-002 | Multi-Tenant | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-DB-001 | Database | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-DB-002 | Database | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-DB-003 | Database | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-DB-004 | Database | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-DB-005 | Database | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-DB-006 | Database | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-DB-007 | Database | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-DB-008 | Database | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-DB-009 | Database | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-BE-001 | Backend | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-BE-002 | Backend | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-BE-003 | Backend | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-BE-004 | Backend | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-BE-005 | Backend | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-BE-006 | Backend | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-BE-007 | Backend | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-BE-008 | Backend | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-SI-001 | Sales & Inventory | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-SI-002 | Sales & Inventory | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-SI-003 | Sales & Inventory | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-SI-004 | Sales & Inventory | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-SI-005 | Sales & Inventory | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-SI-006 | Sales & Inventory | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-SI-007 | Sales & Inventory | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-SI-008 | Sales & Inventory | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-SI-009 | Sales & Inventory | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-SI-010 | Sales & Inventory | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-SI-011 | Sales & Inventory | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-SI-012 | Sales & Inventory | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-SI-013 | Sales & Inventory | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| FIN-001 | Finance | APPROVED (label retained; provenance UNVERIFIED) | Round 2 audited snapshot 184b32f7 | Find explicit user approval; do not change original ADR |
| FIN-002 | Finance | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-CRM-001 | CRM | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-CRM-002 | CRM | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-CRM-003 | CRM | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-CRM-004 | CRM | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-CRM-005 | CRM | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-UX-001 | UX/UI | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-UX-002 | UX/UI | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-UX-003 | UX/UI | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-UX-004 | UX/UI | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-UX-005 | UX/UI | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-UX-006 | UX/UI | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-MOB-001 | Mobile | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-MOB-002 | Mobile | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-MOB-003 | Mobile | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-MOB-004 | Mobile | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-MOB-005 | Mobile | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-MOB-006 | Mobile | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-MOB-007 | Mobile | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-MOB-008 | Mobile | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-MOB-009 | Mobile | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-SEC-001 | Security | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-SEC-002 | Security | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-SEC-003 | Security | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-SEC-004 | Security | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-SEC-005 | Security | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-SEC-006 | Security | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-SEC-007 | Security | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-SEC-008 | Security | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-SEC-009 | Security | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-SEC-010 | Security | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-INFRA-001 | Infrastructure | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-INFRA-002 | Infrastructure | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-INFRA-003 | Infrastructure | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-INFRA-004 | Infrastructure | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-INFRA-005 | Infrastructure | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-INT-001 | Integrations | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-INT-002 | Integrations | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-INT-003 | Integrations | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-INT-004 | Integrations | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-INT-005 | Integrations | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-LOC-001 | Localization | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-LOC-002 | Localization | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-LOC-003 | Localization | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |
| ADR-LOC-004 | Localization | PROPOSED | Round 2 audited snapshot 184b32f7 | Check owner ADR and approval before changing status |

- **FIN-001:** original status **APPROVED remains unchanged**; provenance `UNVERIFIED` (AR-004), neither an endorsement nor silent downgrade. Resolve with explicit original user instruction or user reapproval decision.
- **AR-006:** ADR-MOB-004 and ADR-MOB-009 exist (PROPOSED), not missing. Their open policies are not approved.
- **AR-007:** five central registries exist; remaining synchronization work is not proof of architecture approval.
- **Tech stack, identity/tenant/data ownership, HA and UX numeric gates remain PROPOSED/OPEN.**


## v0.3 — Final cross-module status/provenance delta (2026-10-12)

**Evidence:** [current-main sync baseline](https://github.com/yakhyodev/erp-saas-architecture/commit/f26d4587dcad20f4bf7e8404e27cdcaeace9a355); compared against previous Master commit `6f6c7e9bb04feaf9a586bfcc964110eef6a79084`: 83 commits ahead, 3 added ADR files, multiple peer ADR/module refinements. Round 2 historical audited snapshot still `184b32f7b3dcc9ce8d329ca2dfb7895f06cae905`. The prior 97-row status/provenance index above is retained; these entries **extend it to 100 known ADRs** rather than reprinting older rows. Existing changed ADRs were checked for current status where amended: they remain PROPOSED except FIN-001 retaining its historical APPROVED label. This is a status index, not approval certification.

| New ADR ID | Owner | Current file status | Provenance | Governance action |
|---|---|---|---|---|
| [ADR-SI-014](docs/adr/ADR-SI-014.md) | 05 Sales & Inventory | PROPOSED | Current-main ADR, 2026-10-12; AR-002/008 | Resolve Catalog scope and Party writer |
| [ADR-SI-015](docs/adr/ADR-SI-015.md) | 05 Sales & Inventory | PROPOSED | Current-main ADR, 2026-10-12; AR-005/010 | Sign owner-local transaction, GL recognition and charge lifecycle |
| [FIN-003](docs/adr/FIN-003.md) | 06 Finance | PROPOSED | Current-main ADR, 2026-10-12; AR-001/008 | Resolve canonical Party, legal debtor, payer remitter |

**Index total (current known inventory): 100 = 99 PROPOSED + FIN-001 APPROVED(label)/UNVERIFIED(provenance).** FIN-001 original status untouched; no evidence of fresh signed explicit approval established. Current modified ADRs retain proposal status even if their parent module records a user-selected candidate for further discussion. **SELECTED_PENDING_APPROVAL is NOT APPROVED.**

**Candidate selections recorded by Billing §21 — all PROPOSED / user selected for review, not approved:** (1) Company 1:1 BillingAccount 1:1 Wallet; (2) anniversary-based **calendar** month (not 30 days); (3) activated Seat + day proration; (4) server-approved business-effective month, closed-period linked current-open adjustment; (5) marginal progressive discount bands. Legal payer, dates, cutoff, charges, stacking, reconciliation and other policy details OPEN. Approved no-estimated-limit/actual-only negative balance invariant unchanged.

AR/MR current owner/status matrix: [OPEN_QUESTIONS v0.3](OPEN_QUESTIONS.md); canonical proposed writer map: [MODULE_REGISTRY v0.3](MODULE_REGISTRY.md). Codex readiness: **NOT REASSESSED**; prior Round 2 BLOCKED is historical audit verdict, not an independent new readiness finding.
