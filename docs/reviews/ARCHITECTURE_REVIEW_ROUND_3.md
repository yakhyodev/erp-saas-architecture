# ERP SaaS — ARCHITECTURE REVIEW ROUND 3

> **Status:** REVIEWED — CORRECTIONS & FORMAL APPROVALS REQUIRED  
> **Independent auditor:** 14 — Architecture Review  
> **Repository:** [yakhyodev/erp-saas-architecture](https://github.com/yakhyodev/erp-saas-architecture); **branch:** main  
> **Frozen audited source HEAD:** [34fbb99db8c71a01dbb50ef71a6f7555a3310e91](https://github.com/yakhyodev/erp-saas-architecture/commit/34fbb99db8c71a01dbb50ef71a6f7555a3310e91)  
> **Prior Round 2 report:** [ARCHITECTURE_REVIEW_ROUND_2.md](ARCHITECTURE_REVIEW_ROUND_2.md) — assessed source HEAD 184b32f7b3dcc9ce8d329ca2dfb7895f06cae905  
> **Original historical baseline:** [016573c8426ed3ea3b1eacbd0dc81adb67861220](https://github.com/yakhyodev/erp-saas-architecture/commit/016573c8426ed3ea3b1eacbd0dc81adb67861220)  
> **Date:** 2026-10-12. This document is based on current files read at the frozen audited source HEAD, before the review-publication commit.  
> **Whole-ERP Codex architecture-readiness verdict:** **BLOCKED**. No application code or unapproved decision authorized.

## 1. Executive Summary

**Independent current GitHub recount: 126 Markdown files = 100 ADRs + 18 module docs + 5 root registers + README + 2 prior reviews.** Fourteen architecture directions (00–13) remain. All **100 individual ADR files, 18 module files, five root governance files, README and two historical audit reports** were read using the GitHub connector at the audited main HEAD. Relevant sections are linked below. Historical Round 2 audited-source count was **121 Markdown / 97 ADR / 17 modules**; Round 2 review creation and three new ADRs plus the provider appendix explain the current difference. The previous Round 2 review itself remains unchanged.

**Change of substance:** Database §6 vs §17 inconsistency is now explicitly corrected at §6 and §18 with owner-local ACID+outbox preferred as PROPOSED. Central MODULE_REGISTRY v0.3 explicitly supersedes older broad membership wording *as a proposal*, and matches Security/Multi-Tenant/Billing current recommended writer split. The SalesDoc provider capability and method evidence appendix is restored as DOCUMENTED, not LIVE_VERIFIED. UX §17 supplies detailed acceptance oracles for all twenty tasks. Five Billing §21 preferences are SELECTED_PENDING_APPROVAL, **never business APPROVED**. New ADR-SI-014, ADR-SI-015 and FIN-003 clarify cross-module data/financial ownership without independently approving it.

**Independent assessment:** 8/13 historical findings have a named documentary sub-gap corrected (AR-002/003/005/006/007/011/012/013), but **0/13 have all relevant approvals and acceptance conditions fully closed**. All MR-01…MR-20 have at least an OPEN business, technical, governance, evidence or test dependency. Active canonical documentation has **no proven APPROVED-vs-APPROVED contradiction**. Priority unresolved issues are account/legal payer/Party/cardinality, negative month lifecycle, chargeability, Finance recognition, membership ownership and non-functional launch decisions. Runtime production security, usability, backup recovery and external API behaviors remain **NOT_TESTED/NOT_LIVE_VERIFIED**, not passed.

**Codex readiness:** 0 READY, 2 READY_WITH_CONDITIONS (UX, Localization), 12 BLOCKED. **Whole ERP = BLOCKED**. “READY” is architecture authorization readiness, not a production pass and never replaces the user’s separate permission to write code.

## 2. Audit Methodology and Inventory

Read-only GitHub verification: retrieve live main HEAD and untruncated recursive tree, inspect every Markdown/ADR individually, compare the Round 2 review and current root governance indexes against current module contracts, identify typed source facts/authoritative writers, distinguish historical variants from current proposals, then evaluate gates independently. All source files are immutable for purposes of this report; link paths below resolve relative to docs/reviews. Code/test execution, live API calls, production DR, backup restore, performance and penetration testing were **not** conducted.

| Source category | Current files | Audit coverage |
|---|---:|---|
| docs/adr | **100** | Master 9; Billing 10; MT 2; DB 9; Backend 8; Sales & Inventory 15; Finance 3; CRM 5; UX 6; Mobile 9; Security 10; Infrastructure 5; Integration 5; Localization 4. |
| docs/modules | **18** | Master 1; Billing 1; Multi-Tenant 1; DB 1; Backend 1; Sales 1; Finance 2; CRM 1; UX 2; Mobile 2; Security 1; Infrastructure 1; Integration 2 incl. newly restored provider appendix; Localization 1. |
| Root Markdown | **6** | Five core registries plus README. |
| docs/reviews before this commit | **2** | Round 1 original and Round 2 complete report retained. |
| **Total before Round 3 publication** | **126** | No missing ADR referenced as present in current tree. |

ADR document status audit: **99 PROPOSED + FIN-001 existing APPROVED label with unverified explicit original user approval provenance**. The 100-row current index is [DECISIONS v0.3](../../DECISIONS.md). Many ADRs contain alternatives, reason, risks and module effects; their design contracts and test matrices must not be mistaken for implemented/proven behavior. For some legacy ADRs the operative recent section refines an older paragraph; current scoped wording prevails *only as a PROPOSED design coordination note*. No ADR was modified or approved by this review.

**Approval vocabulary:** APPROVED = original confirmed user business invariants only, plus a FIN-001 *document label* whose independent provenance is explicitly unresolved; PROPOSED = architectural alternative or default; SELECTED_PENDING_APPROVAL = user selected for discussion, not formal approved policy; FIXED_DOCUMENTATION = a specific former documentation gap resolved; HARMONIZED_PROPOSED = competing proposals aligned but unapproved; OPEN = decision/verification needed; NOT_TESTED = spec written, runtime proof absent; LIVE_VERIFIED only when actual authorized provider test evidence exists.

## 3. Original APPROVED Business Invariant Protection

| ID | Immutable user requirement | Current source / outcome |
|---|---|---|
| BIZ-001 | Hybrid Prepaid SaaS | [DECISIONS](../../DECISIONS.md), Billing §1: **preserved** |
| BIZ-002 | Arzon Seat + Business Usage + Resource Usage | Billing §1/§21: **preserved** |
| BIZ-003 | Business Usage: each eligible document base fee, additional credits above volume threshold | Billing §20.4/§21: **preserved**; eligibility/units OPEN |
| BIZ-004 | **1 credit = 10 so‘m** | Billing §21, Finance: **preserved** |
| BIZ-005 | Progressive volume discounts + enterprise individual contracts | Billing §21: **preserved**; marginal approach not approved |
| BIZ-006 | After balance exhaustion actual Seat/Usage expenses may be posted negative for **one calendar month** | BILL-006, Billing §21: **preserved**, expiry interpretation OPEN |
| BIZ-007 | **No estimated negative credit ceiling** | BILL-006, Billing §20–21: **preserved**; no forecast cap proposed as approved |
| BIZ-008 | Companies and branches on one SaaS platform | MT §19–20: **preserved**; Tenant ≠ necessarily Company |
| BIZ-009 | Uzbek/English/Russian, hidden future Uzbek alphabet variant | Localization §15 and LOC-004: **preserved**, hidden OFF by default PROPOSED |
| BIZ-010 | UX substantially simpler, clearer and easier to learn than Sales Doctor | UX §17: **preserved quality objective**, superiority unmeasured |

**Strict boundary:** Go, PostgreSQL, Next.js, Flutter, modular monolith, RLS, HA topology, shared-schema/dedicated DB, timeouts, SLOs, discount mechanism and Billing §21 selected choices all remain **PROPOSED or OPEN**. No source proves new formal business approvals beyond the original invariants. [DECISIONS](../../DECISIONS.md) and [REQUIREMENTS](../../REQUIREMENTS.md).

**FIN-001 provenance:** FIN-001 still displays APPROVED as historical label, but the original dated user confirmation was **not independently located**. Its appended A/B/C provenance options do not confirm it. **AR-004 remains MISSING_APPROVAL_PROVENANCE; the label is not altered by this review.** [FIN-001](../adr/FIN-001.md), [DECISIONS](../../DECISIONS.md).

## 4. AR-001…AR-013 — Round 2 vs Current Evidence Matrix

Status may be multi-valued. “FIXED_DOCUMENTATION” refers to its documentary sub-gap, not completion of every issue. Severity measures *potential design/approval impact*, not a proven production defect.

| ID | Severity | Current status | Old evidence → Current evidence / audit finding | Responsible chats | Required next action | Current source |
|---|---|---|---|---|---|---|
| **AR-001** | HIGH | HARMONIZED_PROPOSED; SELECTED_PENDING_APPROVAL | Round 2: company wallet vs tenant/account ambiguity. Current Billing §20.1/§21 and MT §19–20 preserve company-to-account 1:1 MVP candidate, distinct source attribution/liability and payer. | 01 Billing, 02 Multi-Tenant, 06 Finance | Approve Company/LegalEntity/BillingAccount/Wallet, legal debtor and third-party payer, migration; selected candidate NOT APPROVED. | [BILLING_ARCHITECTURE](../modules/BILLING_ARCHITECTURE.md) §21; [MULTI_TENANT_ARCHITECTURE](../modules/MULTI_TENANT_ARCHITECTURE.md) §20; [ADR-BILL-001](../adr/ADR-BILL-001.md) |
| **AR-002** | HIGH | FIXED_DOCUMENTATION; HARMONIZED_PROPOSED; MISSING_APPROVAL | Round 2: Sales vs Catalog Product master dispute. Current Catalog solely proposes canonical Product/SKU/UoM writer; Pricing, Sales, Inventory and Purchasing distinct; CRM old broad wording is historical proposal. | 04 Backend, 05 Sales, 07 CRM, 00 Master | Approve tenant/global/company Catalog namespace, module packaging and Pricing applicability; no alternate Product writer. | [ADR-SI-014](../adr/ADR-SI-014.md); [ADR-CRM-001-domain-ownership](../adr/ADR-CRM-001-domain-ownership.md); [BACKEND_ARCHITECTURE](../modules/BACKEND_ARCHITECTURE.md) §18 |
| **AR-003** | MEDIUM | FIXED_DOCUMENTATION; HARMONIZED_PROPOSED; MISSING_APPROVAL | Round 2: Stage A/B reconciled. Current DB §18.5 and Infrastructure §19 both distinguish Stage A single primary+PITR NOT HA, Stage B standby/fencing, Stage C measured auto failover. | 03 Database, 11 Infrastructure | Approve initial A or B, recovery region, retention, uptime SLO, RPO/RTO, budget; design recovery drills. | [DATABASE_ARCHITECTURE](../modules/DATABASE_ARCHITECTURE.md) §18.5; [INFRA_ARCHITECTURE](../modules/INFRA_ARCHITECTURE.md) §19; [ADR-DB-007](../adr/ADR-DB-007.md) |
| **AR-004** | HIGH | MISSING_APPROVAL | Round 2: FIN-001 APPROVED status had no independent user provenance. Current FIN-001 added explicit A/B/C provenance packet; label deliberately unchanged, original dated approval still unverified. | 06 Finance, 00 Master | Find original dated explicit user decision or obtain fresh re-confirmation; do NOT auto-downgrade/upgrade or claim historical proof. | [FIN-001](../adr/FIN-001.md); [DECISIONS](../../DECISIONS.md) v0.3; [FINANCE_ARCHITECTURE](../modules/FINANCE_ARCHITECTURE.md) |
| **AR-005** | HIGH | FIXED_DOCUMENTATION; HARMONIZED_PROPOSED; MISSING_APPROVAL | Round 2: DB §6 vs §17 cross-domain ACID ambiguity. Current DB §6 explicitly supersedes its older recommendation as historical PROPOSED alternative; §18 and ADR-DB-004 favor owner-local ACID+outbox, with per-fact ownership and reconciliation. | 03 Database, 04 Backend, 05 Sales/Inventory, 06 Finance | Obtain technical ADR sign-off for owner-local ACID/outbox; approve GL/revenue/COGS/tax triggers and dispatch/invoice/reversal policies; no duplicate physical posting. | [DATABASE_ARCHITECTURE](../modules/DATABASE_ARCHITECTURE.md) §6/§17/§18; [ADR-DB-004](../adr/ADR-DB-004.md); [ADR-SI-015](../adr/ADR-SI-015.md); [FIN-002](../adr/FIN-002.md) |
| **AR-006** | MEDIUM | FIXED_DOCUMENTATION; HARMONIZED_PROPOSED; POLICY_OPEN | Round 2: missing MOB-004/009 already restored. Current Mobile/SEC add detailed operation_id retry, UNKNOWN/conflict, device/session revoke and scope revalidation; remote wipe while offline NOT guaranteed. | 09 Mobile, 10 Security, 04 Backend | Sign offline TTL, refresh, quarantine/deletion/legal hold, command conflict owners and revoke freshness; execute security tests later. | [ADR-MOB-004](../adr/ADR-MOB-004.md); [ADR-MOB-009](../adr/ADR-MOB-009.md); [MOBILE_ARCHITECTURE](../modules/MOBILE_ARCHITECTURE.md) Round 2; [SECURITY_ARCHITECTURE](../modules/SECURITY_ARCHITECTURE.md) §21 |
| **AR-007** | MEDIUM | FIXED_DOCUMENTATION; GOVERNANCE_SYNC_OPEN | Round 2: five global registries exist. Current DECISIONS v0.3 indexes 100 ADR (99 PROPOSED + FIN-001 label with unverified provenance); OPEN_QUESTIONS v0.3 tracks all 13 AR and 20 MR; historical sections retained and labeled superseded. | 00 Master, 14 Audit | Maintain versioned index and per-ADR source approval evidence; fix root-file relative source links where broken; current audit does not edit registries. | [DECISIONS](../../DECISIONS.md) v0.3; [OPEN_QUESTIONS](../../OPEN_QUESTIONS.md) v0.3; [MODULE_REGISTRY](../../MODULE_REGISTRY.md) v0.3 |
| **AR-008** | HIGH | HARMONIZED_PROPOSED; MISSING_APPROVAL | Round 2: Party/Counterparty/Payer/Outlet conflated. Current CRM/FIN-003 distinguish Party vs Customer/Outlet, Finance LegalDebtor/ReceivableOwner, third-party Remitter and independent SaaS payer; Party authoritative writer still CRM vs shared context. | 07 CRM, 06 Finance, 05 Sales, 12 Integrations | Select one tenant-scoped Party master, legal snapshot/merge/split, cross-company settlement and verified third-party payer authorization. | [FIN-003](../adr/FIN-003.md); [CRM_DISTRIBUTION](../modules/CRM_DISTRIBUTION.md); [ADR-INT-005](../adr/ADR-INT-005.md) |
| **AR-009** | HIGH | HARMONIZED_PROPOSED; SELECTED_PENDING_APPROVAL | Round 2: offline month lacked anchor. Current Billing §21 records server-approved domain-effective period + current-open-period linked adjustment for closed month as candidate; Mobile timestamps separated; one calendar-month negative rule unchanged. | 01 Billing, 09 Mobile, 04 Backend, 05 Sales | Approve trusted effective time and cutoffs, posted/economic periods, invoice corrections, tier/version and negative grace interaction, timezone and month expiry. | [BILLING_ARCHITECTURE](../modules/BILLING_ARCHITECTURE.md) §20.5/§21; [ADR-MOB-008](../adr/ADR-MOB-008.md); [ADR-BILL-003](../adr/ADR-BILL-003.md) |
| **AR-010** | HIGH | HARMONIZED_PROPOSED; MISSING_APPROVAL | Round 2: document charge trigger unspecified. Current Billing §20.4/§21, Sales §12–13 and BILL-009/SI-013 separate non-monetary fact from Billing economic occurrence and cover partials, import, return and reversal. | 01 Billing, 05 Sales, 12 Integrations, 06 Finance | Business approves type × lifecycle transition × fee/overage/volume × cancel/refund/amend/import eligibility; semantic occurrence unique. | [ADR-BILL-009](../adr/ADR-BILL-009.md); [ADR-SI-013](../adr/ADR-SI-013.md); [BILLING_ARCHITECTURE](../modules/BILLING_ARCHITECTURE.md) §20.4 |
| **AR-011** | MEDIUM | FIXED_DOCUMENTATION; NOT_LIVE_VERIFIED | Round 2: detailed provider evidence lost in main architecture memo. Current SalesDoc V2 provider appendix restores documented methods, CS_id/SD_id/code_1C, payer modes, pagination, errors, selected method-specific rate limits and migration/cutover. API has NOT been live-tested. | 12 Integrations, 07 CRM, 10 Security | Preserve DOCUMENTED labels, run authorized sandbox/live verification and read-first migration rehearsal prior to outbound/cutover; keep external adapter PROPOSED. | [SALESDOC_V2_PROVIDER_COMPATIBILITY](../modules/SALESDOC_V2_PROVIDER_COMPATIBILITY.md) §10–§100; [INTEGRATION_ARCHITECTURE](../modules/INTEGRATION_ARCHITECTURE.md) R2-DOC-02; [ADR-INT-001](../adr/ADR-INT-001.md) |
| **AR-012** | MEDIUM | FIXED_DOCUMENTATION; HARMONIZED_PROPOSED; NOT_TESTED; MISSING_APPROVAL | Round 2: candidate UX-T01…20 existed. Current UX §17 adds per-task persona, screen sequence, illustrative permission, E/L/X/O states, success/failure oracle and synthetic dataset, plus search/a11y/localization matrices. No pilot or comparative outcome. | 08 UX, 00 Master, 09 Mobile, 13 Localization, domain owners | Approve MVP task/device/role priorities and target baselines; conduct real user/permission/a11y/performance tests during implementation; don't claim Sales Doctor superiority yet. | [UX_ARCHITECTURE](../modules/UX_ARCHITECTURE.md) §16–17; [ADR-UX-004](../adr/ADR-UX-004.md); [ADR-UX-006](../adr/ADR-UX-006.md) |
| **AR-013** | HIGH | FIXED_DOCUMENTATION; HARMONIZED_PROPOSED; MISSING_APPROVAL | Round 2: MODULE_REGISTRY old broad Security membership wording differed from MT/Security. Current registry v0.3 expressly supersedes old wording AS PROPOSAL and recommends IdentityAuth→Security, TenantMembership→MT, RoleAssignment/PolicyEvaluation→Security, enforcement→Backend. Billing-010 similarly clarifies historical wording; alternative Security-owned membership remains OPEN. | 00 Master, 02 Multi-Tenant, 04 Backend, 10 Security, 01 Billing | Approve the one-writer model, tenant switch, grant scope, deny-on-stale, revoke/version SLA and tested RLS/worker integration; avoid dual membership writers. | [MODULE_REGISTRY](../../MODULE_REGISTRY.md) v0.3; [SECURITY_ARCHITECTURE](../modules/SECURITY_ARCHITECTURE.md) §21; [ADR-MT-002](../adr/ADR-MT-002.md); [ADR-BILL-010](../adr/ADR-BILL-010.md) |

**Reconciliation outcome:** Eight specific prior documentation defects now have corrected written treatment (AR-002/003/005/006/007/011/012/013). AR-001/008/009/010 contracts are substantially harmonized but involve unresolved economic or identity choices; AR-004 remains provenance-unverified. These counts do not mean eight issues are fully closed. **Current canonical unresolved owner proposals are not misreported as user-approved architecture conflicts.**

## 5. MR-01…MR-20 — Current Requirements & Evidence Matrix

All 20 tracked MR remain not fully closed; distinguish **business approval**, **technical ADR sign-off** and **tests to execute later**. Detailed owners and dependency history are in [OPEN_QUESTIONS v0.3](../../OPEN_QUESTIONS.md). The column “Current evidence” is design documentation, **not test evidence**.

| MR | Current assessment | What remains | Owner chats | Current evidence |
|---|---|---|---|---|
| MR-01 | SELECTED_PENDING_APPROVAL / BUSINESS | Company 1:1 BillingAccount 1:1 Wallet is only candidate; legal entity/payer/debt holder open. | 01/02/06 | [BILLING_ARCHITECTURE](../modules/BILLING_ARCHITECTURE.md) §21 |
| MR-02 | SELECTED_PENDING_APPROVAL / BUSINESS | Anniversary-based calendar month candidate; clamp vs roll-forward, timezone, first-negative, partial refill, repeat and expiry open. | 01 | [ADR-BILL-006](../adr/ADR-BILL-006.md) |
| MR-03 | SELECTED_PENDING_APPROVAL / BUSINESS | Activated Seat with day proration candidate; suspension, duplicate branch/company seats, dates and tariff open. | 01/02/10 | [ADR-BILL-010](../adr/ADR-BILL-010.md) |
| MR-04 | SPECIFIED_PROPOSED / BUSINESS OPEN | Document types/status/chargeable occurrence plus volume and Resource Usage units/rates/thresholds not signed. | 01/05/12 | [ADR-BILL-009](../adr/ADR-BILL-009.md) |
| MR-05 | SELECTED_PENDING_APPROVAL / BUSINESS | Marginal progressive discount candidate; tiers, window, enterprise precedence/stacking and rounding OPEN. | 01 | [ADR-BILL-007](../adr/ADR-BILL-007.md) |
| MR-06 | SPECIFIED_PROPOSED / BUSINESS-LEGAL OPEN | Jurisdiction, tax point, legal entity books, functional currency, FX, fiscal period and financial recognition open. | 06 | [FINANCE_ARCHITECTURE](../modules/FINANCE_ARCHITECTURE.md) Round 2 |
| MR-07 | FIXED_DOCUMENTATION / TECH SIGN-OFF + BUSINESS OPEN | Typed Sales/Inventory/Finance posting and owner-local ACID+outbox reconciled; GL/COGS/revenue triggers open. | 03/04/05/06 | [DATABASE_ARCHITECTURE](../modules/DATABASE_ARCHITECTURE.md) §6/§18; [ADR-SI-015](../adr/ADR-SI-015.md) |
| MR-08 | SPECIFIED_PROPOSED / BUSINESS OPEN | Negative stock, reservation TTL, valuation FIFO/average, count cutoff, partial delivery and reversal open. | 05/06 | [SALES_INVENTORY](../modules/SALES_INVENTORY.md) §12.6 |
| MR-09 | SPECIFIED_PROPOSED / OWNERSHIP OPEN | Canonical Party CRM vs shared; merge/split, debtor snapshots, third-party payer/legal role open. | 07/06/12 | [FIN-003](../adr/FIN-003.md) |
| MR-10 | SPECIFIED_PROPOSED / TECH SIGN-OFF | Versioned API/command/event IDs, event-vs-occurrence dedup, compatibility and retention proposed; real contract sign-off absent. | 04/12 | [BACKEND_ARCHITECTURE](../modules/BACKEND_ARCHITECTURE.md) §18; [INTEGRATION_ARCHITECTURE](../modules/INTEGRATION_ARCHITECTURE.md) |
| MR-11 | SPECIFIED_PROPOSED / TECH SIGN-OFF + TEST NOT RUN | Shared schema/RLS optional; pooled transaction context, role bypass, cache/workers/search, tenant placement negative tests unrun. | 02/03/10 | [DATABASE_ARCHITECTURE](../modules/DATABASE_ARCHITECTURE.md) §18.3; [SECURITY_ARCHITECTURE](../modules/SECURITY_ARCHITECTURE.md) §21.4 |
| MR-12 | FIXED_DOCUMENTATION / SECURITY POLICY OPEN | Offline ADRs and state machine documented; session TTL, revoke, local data quarantine, conflict operator and retention open. | 09/10/04 | [ADR-MOB-004](../adr/ADR-MOB-004.md); [ADR-MOB-009](../adr/ADR-MOB-009.md) |
| MR-13 | SPECIFIED_PROPOSED / PRIVACY BUSINESS OPEN | GPS lawful basis, visit proof, retention, legal hold, geofence threshold, consent/withdrawal open. | 07/09/10 | [ADR-CRM-002-offline-gps-privacy](../adr/ADR-CRM-002-offline-gps-privacy.md) |
| MR-14 | HARMONIZED_PROPOSED / BUSINESS SLA OPEN + NOT_TESTED | Stage A/B/C coherent; initial HA choice, RPO/RTO, backup retention/region, DR budget unapproved; restore not run. | 03/11 | [INFRA_ARCHITECTURE](../modules/INFRA_ARCHITECTURE.md) §19 |
| MR-15 | SPECIFIED_PROPOSED / PERF TEST NOT_RUN | Pool/resource budgets and hot-tenant/month-end/stock/sync load plans exist; targets and benchmark unmeasured. | 11/03/04 | [INFRA_ARCHITECTURE](../modules/INFRA_ARCHITECTURE.md) §19.3–19.5 |
| MR-16 | FIXED_DOCUMENTATION / MVP BUSINESS OPEN + NOT_TESTED | Detailed UX-T01…20 persona/screens/permissions/states/dataset matrix exists, priorities and field pilot not approved. | 08/00/all domains | [UX_ARCHITECTURE](../modules/UX_ARCHITECTURE.md) §17.2 |
| MR-17 | SPECIFIED_PROPOSED / SECURITY TEST NOT_RUN | Authorization across global search results/counts/facets/recents/index/cache/reindex and locale switch designed, not tested. | 08/10/13/03 | [LOCALIZATION_ARCHITECTURE](../modules/LOCALIZATION_ARCHITECTURE.md) §15.3; [UX_ARCHITECTURE](../modules/UX_ARCHITECTURE.md) §17.5 |
| MR-18 | BUSINESS MVP SCOPE OPEN | HR, Manufacturing, Purchasing, Reporting inclusion/exclusion and phases undecided. | 00/business | [OPEN_QUESTIONS](../../OPEN_QUESTIONS.md) v0.3 |
| MR-19 | FIXED_DOCUMENTATION / LIVE_NOT_VERIFIED | SalesDoc provider methods/error/rate-limit/migration source evidence restored DOCUMENTED, live auth/production/cutover unverified. | 12/10 | [SALESDOC_V2_PROVIDER_COMPATIBILITY](../modules/SALESDOC_V2_PROVIDER_COMPATIBILITY.md) §10–§100 |
| MR-20 | SPECIFIED_PROPOSED / UX BUSINESS OPEN + NOT_TESTED | Device/network/locale/a11y matrix, WCAG and LCP/INP/CLS targets proposed; no signed thresholds, user pilot or automated test. | 08/09/13 | [UX_ARCHITECTURE](../modules/UX_ARCHITECTURE.md) §17.7–17.8; [LOCALIZATION_ARCHITECTURE](../modules/LOCALIZATION_ARCHITECTURE.md) §15.6 |

## 6. Cross-Module Contract Consistency and Authoritative Writers

| Domain entity or operation | Proposed single writer / relationship | Alignment and remaining boundary |
|---|---|---|
| Tenant, Company, Branch, org graph | Multi-Tenant | Company/LegalEntity relationship still OPEN; branch scope validated separately from tenant isolation. |
| Global principal, credentials, sessions/device auth | Security Identity/Auth | Backend enforces, never independently writes global credentials. |
| TenantMembership status/version | **Multi-Tenant proposed candidate** | MODULE_REGISTRY v0.3, Security §21, MT §20 and BILL-010 concur as current recommendation. Alternative Security writer remains OPEN until sign-off. |
| RoleAssignment and PolicyDefinition | Security Permissions candidate | Grants and policies versioned; enforce per server object/use-case. |
| PermissionEvaluation | Security evaluation; Backend/domain guard enforcement | UI hiding no authority; revoke/tenant switch/cache freshness OPEN. |
| Catalog/Product/SKU/UoM | Catalog logical writer candidate | Physical module placement/tenant namespace OPEN; Product ID vs SKU identity distinct. |
| PriceList / commercial pricing | Pricing logical writer candidate | Distinct from SaaS Billing tariffs and Inventory stock. |
| SalesOrder and commercial dispatch intent | Sales | Inventory alone mutates StockMovement and StockBalance; invoice not physical posting. |
| Inventory reservation/movement/balance | Inventory | One movement per fact/leg, append-only, local ACID+outbox preferred. |
| ERP GL, AR/AP, invoice/credit note | Finance | Tax/jurisdiction/recognition triggers OPEN; no SaaS Wallet mutation. |
| Canonical Party, Customer/Outlet | Party owner **CRM vs shared Party OPEN**; CRM relationship writer | Finance owns legal debtor obligations and immutable invoice snapshots, not party master. |
| ERP Remitter/Payer and ReceivableOwner | Finance settlement evidence and AR obligation, references Party | Remitter ≠ legal debtor ≠ outlet; third-party payment allocation requires approval. |
| SaaS BillingAccount/Wallet/rating | Billing | Company 1:1 BillingAccount 1:1 Wallet selected pending approval; SaaS payer independent of ERP debtor. |
| Provider external ID & evidence | Integrations mapping adapter | CS_id/SD_id/code_1C not canonical ERP IDs; domain owner accepts/imports effects. |
| Offline local queue vs accepted command | Mobile local, Backend server-accepted/domain writer | UNKNOWN timeout, reauth on replay, no device wallet/GL/stock authority. |
| Locales, translation, derived aliases | Localization for catalogs/aliases; business owner immutable originals | All read/search surfaces require permission checks; hidden alphabet private. |

Sources: [MODULE_REGISTRY v0.3](../../MODULE_REGISTRY.md); [Backend §18–19](../modules/BACKEND_ARCHITECTURE.md), [Multi-Tenant §20](../modules/MULTI_TENANT_ARCHITECTURE.md), [Security §21](../modules/SECURITY_ARCHITECTURE.md), [ADR-SI-014](../adr/ADR-SI-014.md), [FIN-003](../adr/FIN-003.md), [Billing §21](../modules/BILLING_ARCHITECTURE.md).

### 6.1 Independent decision on Database §6/§17

**VERIFIED FIXED_DOCUMENTATION (not approved technical selection).** [Database §6](../modules/DATABASE_ARCHITECTURE.md) directly explains and explicitly marks its old one-ACID SalesDocument+StockMovement+Finance journal recommendation as “SUPERSEDED AS A RECOMMENDATION (historical PROPOSED alternative only)”. [Database §17–18](../modules/DATABASE_ARCHITECTURE.md) supplies the preferred owner-local ACID transaction + outbox/inbox matrix and its failure/replay, reversal and reconciliation tradeoffs; [ADR-DB-004](../adr/ADR-DB-004.md) matches, [ADR-BE-003](../adr/ADR-BE-003.md), [ADR-SI-015](../adr/ADR-SI-015.md), [FIN-002](../adr/FIN-002.md) corroborate. **No longer two simultaneous normative recommendations.** Yet the model and financial triggers require explicit cross-module technical/domain sign-off before code.

Typed command/invariant outline (PROPOSED): confirmation updates Sales only; reservation affects available stock only; dispatch intent links to Inventory physical posting once; invoice never posts another physical leg; Finance GL journals commit balanced lines atomically under its own rules; Billing accepts separately approved chargeable business occurrence and rates actual credits once. Envelope event_id is distinct from canonical business_occurrence_id; same key+same payload replay harmless, same key+different payload conflict; at-least-once transport, idempotent economic effect; no distributed global commit guarantee. [Backend §18.3–18.5](../modules/BACKEND_ARCHITECTURE.md), [Sales §12](../modules/SALES_INVENTORY.md).

### 6.2 Independent decision on MODULE_REGISTRY membership ownership

**Current v0.3 coordination is aligned as HARMONIZED_PROPOSED, not APPROVED.** [MODULE_REGISTRY v0.3](../../MODULE_REGISTRY.md) expressly says its current writer/dependency table **supersedes older broad “Security owns membership” descriptions as proposal wording only**; proposes TenantMembership writer Multi-Tenant, IdentityAuth/Credentials writer Security Identity, RoleAssignment+PolicyDefinition writer Security Permissions and enforcement Backend/domain guard. [Security §21](../modules/SECURITY_ARCHITECTURE.md), [Multi-Tenant §20](../modules/MULTI_TENANT_ARCHITECTURE.md), [ADR-MT-002](../adr/ADR-MT-002.md), [ADR-BILL-010](../adr/ADR-BILL-010.md) all disclose the same candidate and historical competing phrasing. Thus **historical wording still appears, but the operative proposed source-of-truth map is no longer internally contradictory**. Alternative ownership and revocation SLA remain OPEN; no dual membership writers should be implemented.

## 7. Billing, Finance, Sales/Stock and Offline Integrity

**APPROVED invariants protected:** 1 credit=10 UZS, actual Seat+Business/Resource Usage charge, one calendar month negative, no estimated credit limit. **Not approved details:** threshold values/prices, payout, tax, invoice/credit-note timing, service expiry, refund and interest/late-fee behavior. Do not impose hidden caps or automatic shutdown at expiry. [Billing §20–21](../modules/BILLING_ARCHITECTURE.md), [ADR-BILL-006](../adr/ADR-BILL-006.md).

- **Account/payer:** Company→BillingAccount→Wallet **1:1 candidate for MVP**, not legal payer approval. Source Company/Branch attribution and Billing liability/funder must be snapshotted separately; tenant ERP receivable legal debtor and SaaS Wallet payer are distinct. [ADR-BILL-001](../adr/ADR-BILL-001.md), [FIN-003](../adr/FIN-003.md).
- **Month:** Billing §21 records **anniversary-based calendar-month candidate**, not named calendar month or rolling 30 days. Jan 31→Feb end depends on unapproved clamp/roll-forward. Trigger, timezone, top-up reset, repeated deficit, expiry and post-expiry rights OPEN.
- **Seat:** Activated Seat + day-level prorating SELECTED_PENDING_APPROVAL; exact status (invite/suspend/reactivate), per-company vs per-tenant counting, effective dates and billable interval keys remain OPEN.
- **Late sync:** Device time untrusted; server-authorized business-effective month plus linked adjustment in current open posting month for closed historical period SELECTED_PENDING_APPROVAL. Verify original versus posting period, marginal tier, negative grace and immutable invoice. Mobile does not determine charge month.
- **Chargeable docs:** Sales/Inventory/Finance source lifecycle emits non-monetary facts; Billing alone chooses eligible document type/transition/overage and posts economic effect. For Sales Order/Invoice, Goods Receipt, Purchase Order, Transfer, Return, Count Adjustment and historical imported docs, create/submit/confirm/dispatch/post/finalize, partial/refund/cancel/amendment all OPEN. **Do not infer that physical dispatch or Finance GL posting is itself a platform charge.**
- **Stock/GL:** Confirm does not decrement physical stock; Inventory dispatch movement and balance once; Invoice never decrements again; Finance AR/revenue/COGS/valuation postings require jurisdiction and accounting policy. Owner-local atomic transaction+outbox; immutable reversal with original references, cross-domain reconciliation. [FIN-002](../adr/FIN-002.md), [ADR-SI-015](../adr/ADR-SI-015.md).
- **Ledger separation:** SaaS Billing ledger/wallet != ERP GL/AR/AP, and FIN-001 original approval provenance stays unverified.
- **Import:** Integrated source IDs/revisions and charge occurrence semantics prevent silently double-billing imported history. Chargeability remains a business decision. [Integration provider appendix](../modules/SALESDOC_V2_PROVIDER_COMPATIBILITY.md).

**Formal business blockers:** MR-01…09 and AR-001/004/005/008/009/010 plus FIN jurisdiction/finance recognition; all require precise user and domain owner approvals before money/stock/accounting implementation.

## 8. Security, Tenant Isolation, Performance and HA Quality Gates

**Security design only, NOT TESTED:** Security §21 SEC-R2-01…13 specifies forged IDs, cross-tenant/branch list/detail/write/search/count/facets, RLS USING/WITH CHECK, pooled A→B reuse/rollback, runtime BYPASSRLS/SECURITY DEFINER, worker/outbox, object URLs, revoked sessions/offline device, impersonation and stale placement. Default deny and trusted scope from current Membership+Security; no token/user-entered tenant ID grants unilateral access. Privileged maintenance paths separated, audit records original actor and acting-as. [Security §21](../modules/SECURITY_ARCHITECTURE.md), [Database §18.3](../modules/DATABASE_ARCHITECTURE.md), [ADR-MT-001](../adr/ADR-MT-001.md).

**HA reconciled PROPOSED:** Stage A single PostgreSQL primary, encrypted WAL/PITR and verified restore (NOT HA); Stage B standby, single writer/fencing/failover drills; Stage C optional measured automation. DB §18.5 and Infrastructure §19 match. Initial A/B, RPO/RTO, uptime SLO/SLA, region, backup retention and operating budget require user decision. No restore/failover drill actually executed. [ADR-DB-007](../adr/ADR-DB-007.md), [INFRA §19](../modules/INFRA_ARCHITECTURE.md).

**Performance:** Resource budget/DB pool equation, worker backlog, cache/search/DB IO, object-storage cost, tenant fairness and hot stock/billing month-end/offline burst/load test matrix are documented. p95/p99 thresholds, actual workload, connection budget and benchmark results remain UNAPPROVED/UNMEASURED. Do not prematurely mandate Kafka/Kubernetes/sharding. [Infra §19.3–19.5](../modules/INFRA_ARCHITECTURE.md), [ADR-INFRA-001](../adr/ADR-INFRA-001.md), [ADR-DB-006](../adr/ADR-DB-006.md).

**Test phase distinction:** architecture can approve precise negative/chaos/performance **test specifications** now, but measured passing results belong to implementation/pre-production gates after code/environment exists. Do not call these tests passed or production ready.

## 9. SalesDoc Provider Evidence and Mobile Security

**R2-DOC-02 FIXED_DOCUMENTATION:** a new [SALESDOC_V2_PROVIDER_COMPATIBILITY.md](../modules/SALESDOC_V2_PROVIDER_COMPATIBILITY.md) restores baseline source-specific method/capability matrix, JSON POST /api/v2 with method dispatch, login/auth envelope, pagination GET defaults and SET batches, CS_id/SD_id/code_1C scoped identity, normal vs contragent server payer/outlet rules, errors 400/401/402/403/407/429, selected documented rate limits (e.g., setStock and getStockForDate), correction/import/cutover/reconciliation evidence. Source citations point to published SalesDoc V2 documentation. **Every method/limit is DOCUMENTED and runtime NOT_VERIFIED; 0 LIVE_VERIFIED claims.** External writes remain disabled until authorized verified adapter/migration evidence. [Integration architecture R2-DOC-02](../modules/INTEGRATION_ARCHITECTURE.md), [ADR-CRM-003](../adr/ADR-CRM-003-salesdoc-adapter.md).

**Offline:** [MOBILE_ARCHITECTURE](../modules/MOBILE_ARCHITECTURE.md), [MOBILE_OFFLINE](../modules/MOBILE_OFFLINE.md), MOB-004/009 clearly separate DRAFT→QUEUED→SENDING→ACCEPTED/REJECTED/CONFLICT/UNKNOWN, separate downstream SERVER_EFFECT_PENDING/COMPLETED, resync cursor and server identity/branch/grants/device recheck on each accepted command. Stable operation_id and canonical business occurrence prevent duplicate effect. Offline remote wipe **cannot be guaranteed**. Session TTL, retention/quarantine and GPS lawful basis OPEN. No real mobile device/replay test run.

## 10. UX/UI and Localization — Compulsory Competitive Quality Gate

**Detailed acceptance specification now present (FIXED_DOCUMENTATION only):** [UX §16–17](../modules/UX_ARCHITECTURE.md) names UX-T01…UX-T20 with personas, screen paths, illustrative permission keys (not approved grants), E=empty / L=loading / X=error / O=offline state oracle, positive/forbidden outcomes and synthetic fixtures including separate tenants/branches, revoked member, duplicate IDs and large datasets. Task first dashboard, visible All Functions, novice/advanced mode, command palette and role-specific daily actions are PROPOSED. No cross-domain state can be marked final merely on UI/offline evidence.

| UX test groups | Candidate workflows |
|---|---|
| UX-T01–T03 | Create/find sale, cancel/return |
| UX-T04–T07 | Goods receipt, transfer, stock count, low-stock |
| UX-T08–T11 | Customer create/find, visit, offline order, delivery evidence |
| UX-T12–T15 | AR payment, AP payment, bank reconciliation, financial report |
| UX-T16–T18 | Director cross-branch KPI, sales exception, member permissions |
| UX-T19–T20 | SaaS wallet charges/history, top-up pending vs verified settlement |

**Global search:** Results AND exact-ID detail, counts, autocomplete, facets, snippets, history/recents, caches, exports, reindex and in-flight scope switches require backend current authorization; no tenant side-channel. Search index owner, freshness SLO and auth-version policy OPEN. [UX §17.5](../modules/UX_ARCHITECTURE.md), [Localization §15.3](../modules/LOCALIZATION_ARCHITECTURE.md).

**a11y/localization:** Public uz/en/ru; hidden future alphabet default OFF. Stable semantic translation keys/error codes, versioned compatible web/mobile/offline bundles, Unicode NFC and reversible Uzbek Latin/Cyrillic search aliases; originals and identifiers/money/SKU/INN/document numbers/event IDs unchanged. WCAG 2.2 AA, keyboard/screen-reader/zoom/reflow, long Russian text, mobile touch, UX pilot and safe locale/search collision tests are documented PROPOSED, **not passed**. Hidden standard publication requires future legal approval. [Localization §15](../modules/LOCALIZATION_ARCHITECTURE.md), [ADR-LOC-003](../adr/ADR-LOC-003.md), [ADR-LOC-004](../adr/ADR-LOC-004.md).

**Quantitative targets remain PROPOSED, NOT measured or user-approved:** ≤2–3 deliberate interactions for task **discovery**, not multi-step completion; LCP p75≤2.5s, INP≤200ms, CLS≤0.1, authorized search p75≤1s pending defined hardware/network/dataset. Real observed completion/failure, time, accessibility and comparable Sales Doctor baseline needed before competitive superiority claim. MVP task priorities/feature-device matrix MR-16/20 remain OPEN. **UX design acceptance quality: READY_WITH_CONDITIONS as a specification, not actual usable product certified.**

## 11. Current Design Risk Register and Four Closure Types

| Risk | Severity | Documentation status | Who must close | Closure type |
|---|---|---|---|---|
| Liability/wallet/calendar/seat/discount/chargeable document | HIGH | Harmonized candidate and selected preferences | 01+02+05+06+user | **BUSINESS APPROVAL** |
| Party legal debtor/payer and Catalog scope | HIGH | Proposed single writers, alternative scope OPEN | 04+05+06+07+12+user | **BUSINESS/OWNERSHIP APPROVAL** |
| Finance recognition / tax / valuation / period | HIGH | Typed matrix, no jurisdiction policy approval | 05+06+user/accounting owner | **BUSINESS+LEGAL APPROVAL** |
| Identity/Membership/RoleAssignment single writer | HIGH | Canonical proposal now aligned, old broad wording historical | 02+04+10+00 | **TECH ADR + business scope sign-off** |
| DB source-owner ACID/outbox vs historical one ACID | HIGH impact, DOCUMENTATION_FIXED | One preferred PROPOSED boundary; finance effects still OPEN | 03+04+05+06 | **TECH ADR / account policy sign-off** |
| Cross-tenant/branch/RLS/cache/worker privacy | HIGH potential impact | Negative test designs only | 02+03+10+09 | **TECH ADR + IMPLEMENTATION TEST** |
| HA/DR/connection/scaling | MEDIUM–HIGH | Staged models match, budgets/test designs only | 03+11+user | **SLA BUSINESS+TECH+PREPROD TEST** |
| SalesDoc V2 API/migration/cutover | MEDIUM–HIGH | Historical provider evidence RESTORED, DOCUMENTED | 12+10+owners | **TECH SIGN-OFF + LIVE/API TEST** |
| UX 20 tasks/search/a11y/localization | HIGH strategic, not proof of failure | Full acceptance fixtures documented PROPOSED | 08+09+13+domain owners+user | **UX SCOPE/GOAL APPROVAL + IMPLEMENTATION PILOT** |
| FIN-001 dated provenance | HIGH governance | Historical APPROVED label unverified | 06+00+user | **APPROVAL SOURCE / RECONFIRMATION** |

**New minor documentation hygiene item R3-DOC-01 (LOW):** some root [OPEN_QUESTIONS](../../OPEN_QUESTIONS.md) historical MR evidence links use ../modules/… and ../adr/… paths, which from repository root do not resolve under docs; they should be docs/modules/… and docs/adr/…. v0.3 tracking content remains authoritative in meaning, but clickable evidence navigation needs correction by 00 Master on separate authorization. This is **not** a reason to declare approved contracts incorrect.

**Not an audit finding:** no runtime exploit, data breach, passed load test, failed restore, tax compliance determination or successful production SalesDoc call was observed.

## 12. User Decisions and Technical Sign-offs Needed (priority proposal; NOT APPROVED)

| Priority | Exact decision | Recommended pre-existing PROPOSED choice / qualification | Responsible |
|---|---|---|---|
| P0 | Tenant→Company→LegalEntity→BillingAccount→Wallet, SaaS liability and funder | Company 1:1 BillingAccount 1:1 Wallet MVP candidate; distinguish contracted debtor, payer and source | 01+02+06+user |
| P0 | Negative period | Anniversary **calendar** month candidate (not rolling 30d), decide clamp/roll, timezone, first-negative, top-up/reset, expiry/repeat, post-expiry services | 01+user |
| P0 | Seat + document/resource charges | Activated Seat/day-proration candidate; separately approve seat scope, billable document type+transition, resource unit, threshold/rate and refund/import rules | 01+05+12+10+user |
| P0 | Late event month and discounts | Server-approved effective month, linked current-period closed-month correction candidate; agree price version, marginal tiers, enterprise precedence, invoice+grace implications | 01+04+09+06+user |
| P0 | Finance/tax and inventory | Legal jurisdiction, tax and revenue/COGS point, AR/AP, functional currency, FIFO vs average, reservation TTL/negative stock/count cutoff | 05+06+business |
| P0 | Canonical Party/Payer and Membership owners | One Party master CRM vs shared context; one TenantMembership writer MT vs Security; Security grant owner and Backend guard; choose and version | 00+02+04+06+07+10+user |
| P1 | Catalog scope and MVP | Tenant/global SKU namespace, Pricing module, Purchasing/HR/Manufacturing/Reporting MVP include/exclude | 00+04+05+user |
| P1 | Technology and security | Approve Go/Postgres/Next/Flutter/modular monolith and scoped DB/RLS/worker/token policies only through technical ADR sign-off | 00+03+04+10+11 |
| P1 | Initial production resilience | Stage A vs B, RPO/RTO, uptime SLO, backup region/retention, service budget | 03+11+user |
| P1 | UX quality requirements | Approve UX-T01…T20 priority, persona/device coverage, measured targets and comparative usability pilot | 08+09+13+business |
| P1 | FIN-001 provenance | Locate dated original user approval or explicitly reconfirm boundary without rewriting history | 00+06+user |
| P2 | External SalesDoc, GPS/privacy | No LIVE_VERIFIED; source evidence verification, outbound/cutover authorization, consent/retention/privacy | 07+09+10+12+user |

**No estimate-based credit ceiling, arbitrary extra negative-balance restriction, unapproved fee/threshold or new public locale may be introduced by Codex to fill OPEN decisions.**

## 13. Nine Codex Readiness Gates (Architecture)

| Gate | Current evidence | Verdict | Exit condition |
|---|---|---|---|
| G1 Approval/provenance | DECISIONS v0.3 has 100 status rows; FIN-001 original approval unverified. | **BLOCKED** | Dated user provenance for business/ownership choices; technical ADR approvals. |
| G2 Ownership/cardinality | MODULE_REGISTRY v0.3 one-writer candidates, Billing §21 and Party FIN-003. | **BLOCKED** | BillingAccount/Wallet liability, Catalog scope, Party writer, Membership authority signed. |
| G3 Isolation/security | SEC §21 SEC-R2-01…13 and Database §18.3 design negative suites. | **BLOCKED** | Approve isolation topology and server policy; define reproducible tests before security implementation. |
| G4 Billing integrity | Billing §§20–21, BILL-009/010, actual cost one-calendar month no ceiling. | **BLOCKED** | Approve Seat, type×transition, negative month, late attribution and tariff. |
| G5 Stock/GL/Finance lifecycle | DB §6/18 corrected, SI-015 typed facts, FIN-002/003. | **BLOCKED** | Owner-local ADR sign-off plus accounting recognition/valuation/reversal/business rules. |
| G6 API/offline/external | BE §18, MOB-004/009 and SalesDoc provider appendix. | **BLOCKED** | API/consumer schema versioning, device policy, provider evidence and migration gates. |
| G7 Infrastructure/DR | Infra §19 stages and resource/test matrices, no test results. | **BLOCKED** | Approve launch topology, SLO/RPO/RTO; benchmark/restore evidence by release phase. |
| G8 UX/localization | UX §17 T01…T20, locale §15, E/L/X/O, WCAG target. | **READY_WITH_CONDITIONS (spec only)** | Approve UX priorities and test targets; execute user/a11y/security pilot during implementation. |
| G9 Governance & traceability | Five root registries, 100 ADR index, Round 2 preserved; older rows superseded. | **BLOCKED** | Close material approvals with source links; correct broken links and signed-off ADR/requirements deltas. |

**Test-stage honesty:** implementation-ready architecture requires approved *specs and test obligations*, not fabricated “passed tests”. Runtime pass/fail evidence is collected later in the implementation and controlled preproduction phases. None was executed in this review.

## 14. Module-by-Module Independent Readiness

| Chat | Module | Readiness | Exact remaining blocker / conditions | Evidence |
|---|---|---|---|---|
| 00 | Master Architecture | **BLOCKED** | Canonical owner alternatives and MVP scopes unresolved; formal ADR/user approval provenance. | [MASTER_ARCHITECTURE](../modules/MASTER_ARCHITECTURE.md) §17; [DECISIONS](../../DECISIONS.md) |
| 01 | Monetization & Billing | **BLOCKED** | Billing account/payer, grace month, Seat, document trigger, discounts and usage rates OPEN. | [BILLING_ARCHITECTURE](../modules/BILLING_ARCHITECTURE.md) §20–21 |
| 02 | Multi-Tenant | **BLOCKED** | Membership authority, Company/LegalEntity/BillingAccount and tenant isolation deployment unsigned. | [MULTI_TENANT_ARCHITECTURE](../modules/MULTI_TENANT_ARCHITECTURE.md) §19–20 |
| 03 | Database | **BLOCKED** | §6/§17 document conflict resolved, but owner-local transaction, RLS/HA/DR technical ADRs and tests unsigned. | [DATABASE_ARCHITECTURE](../modules/DATABASE_ARCHITECTURE.md) §6/§18; [ADR-DB-004](../adr/ADR-DB-004.md) |
| 04 | Backend | **BLOCKED** | Auth owner, typed events, stack/API contract and end-to-end transaction sign-off OPEN. | [BACKEND_ARCHITECTURE](../modules/BACKEND_ARCHITECTURE.md) §18–19 |
| 05 | Sales & Inventory | **BLOCKED** | Catalog scope, reservation/valuation, GL trigger and chargeable document lifecycle require decisions. | [SALES_INVENTORY](../modules/SALES_INVENTORY.md) §12–13; [ADR-SI-015](../adr/ADR-SI-015.md) |
| 06 | Finance & Accounting | **BLOCKED** | FIN-001 provenance, Party/debtor, tax/legal, revenue/COGS/AR/AP/fiscal policies open. | [FINANCE_ARCHITECTURE](../modules/FINANCE_ARCHITECTURE.md) Round 2; [FIN-003](../adr/FIN-003.md) |
| 07 | CRM & Distribution | **BLOCKED** | Party writer, payer/outlet relationship, agent and GPS privacy decisions open. | [CRM_DISTRIBUTION](../modules/CRM_DISTRIBUTION.md); [ADR-CRM-001-domain-ownership](../adr/ADR-CRM-001-domain-ownership.md) |
| 08 | UI/UX Architecture | **READY_WITH_CONDITIONS** | Detailed acceptance specification exists; MVP sign-off, persona/role/device profile and measured pilot pending; not unconditional code-ready. | [UX_ARCHITECTURE](../modules/UX_ARCHITECTURE.md) §17 |
| 09 | Mobile & Offline | **BLOCKED** | Offline TTL/device revoke/quarantine and delayed billing month not approved. | [MOBILE_ARCHITECTURE](../modules/MOBILE_ARCHITECTURE.md) Round 2; [ADR-MOB-009](../adr/ADR-MOB-009.md) |
| 10 | Security & Permissions | **BLOCKED** | Membership final ownership, authorization freshness, isolation policy and release negative tests pending. | [SECURITY_ARCHITECTURE](../modules/SECURITY_ARCHITECTURE.md) §21 |
| 11 | Infrastructure & Scaling | **BLOCKED** | Initial Stage A/B, RPO/RTO/SLO, budget, region, restore and failover evidence open. | [INFRA_ARCHITECTURE](../modules/INFRA_ARCHITECTURE.md) §19 |
| 12 | Integrations & SalesDoc API | **BLOCKED** | Documented provider appendix restored; LIVE_VERIFIED absent, Party/import charge/cutover sign-off open. | [SALESDOC_V2_PROVIDER_COMPATIBILITY](../modules/SALESDOC_V2_PROVIDER_COMPATIBILITY.md); [INTEGRATION_ARCHITECTURE](../modules/INTEGRATION_ARCHITECTURE.md) |
| 13 | Localization & Alphabet | **READY_WITH_CONDITIONS** | Immutable originals and locale QA matrix defined; fallback/collisions/search test and hidden rollout not signed/tested. | [LOCALIZATION_ARCHITECTURE](../modules/LOCALIZATION_ARCHITECTURE.md) §15 |

**Numerical result: 12 BLOCKED + 2 READY_WITH_CONDITIONS + 0 READY.** The architecture’s overall implementation-readiness gate is **BLOCKED**. Even after future READY status, explicit separate user permission remains necessary before Codex writes product code.

## 15. Final Audit Verdict, Owner Actions and Evidence

**Current Round 3 independent verdict: REVIEWED — CORRECTIONS & FORMAL APPROVALS REQUIRED; Codex = BLOCKED.**

**Documentary progress is real:** (1) DB §6/§17 conflict now clearly historically superseded as recommendation and owner-local alternative selected only as proposal; (2) Central MODULE_REGISTRY v0.3 membership one-writer candidate is aligned with MT/Security/BILL-010; (3) historical provider-specific API evidence reintroduced; (4) UX-T01…T20 detailed acceptance specifications exist; (5) all 100 ADR indexed, Mobile formerly missing ADRs and five central registries exist. **None of these means corresponding business policy, provider runtime, RLS or accessibility is approved/passed.** New detailed ADR-SI-014/015 and FIN-003 remain PROPOSED.

**Open issues:** 13/13 AR records still have at least a policy/provenance/technical sign-off/test follow-up, despite 8 fixes to discrete documentary subproblems. All 20 MR entries still have remaining approval or verification gates. The next authorized work is P0 business choices, typed ownership/transaction and security ADR sign-offs, then central document synchronization with explicit decisions, MVP scope and UX pilot design. Implementation and real test execution occur only after a separate authorized development phase.

**Historical audits retained:** [Round 1](ARCHITECTURE_REVIEW.md), [Round 2](ARCHITECTURE_REVIEW_ROUND_2.md). **Current immutable evidence source:** [34fbb99db8c71a01dbb50ef71a6f7555a3310e91](https://github.com/yakhyodev/erp-saas-architecture/tree/34fbb99db8c71a01dbb50ef71a6f7555a3310e91). Other modules, ADRs and central registries are **not modified** by this Round 3 review.

### Primary evidence crosswalk

- Central: [DECISIONS](../../DECISIONS.md), [REQUIREMENTS](../../REQUIREMENTS.md), [MODULE_REGISTRY](../../MODULE_REGISTRY.md), [OPEN_QUESTIONS](../../OPEN_QUESTIONS.md), [CHANGELOG](../../CHANGELOG.md).
- Domain contracts: [Master §17](../modules/MASTER_ARCHITECTURE.md), [Billing §21](../modules/BILLING_ARCHITECTURE.md), [MT §20](../modules/MULTI_TENANT_ARCHITECTURE.md), [Database §6/§17/§18](../modules/DATABASE_ARCHITECTURE.md), [Backend §18](../modules/BACKEND_ARCHITECTURE.md), [Sales §12–13](../modules/SALES_INVENTORY.md), [Finance](../modules/FINANCE_ARCHITECTURE.md), [CRM](../modules/CRM_DISTRIBUTION.md), [Security §21](../modules/SECURITY_ARCHITECTURE.md), [Infra §19](../modules/INFRA_ARCHITECTURE.md), [UX §17](../modules/UX_ARCHITECTURE.md), [Localization §15](../modules/LOCALIZATION_ARCHITECTURE.md), [Mobile](../modules/MOBILE_ARCHITECTURE.md), [Integration provider appendix](../modules/SALESDOC_V2_PROVIDER_COMPATIBILITY.md).
- Illustrative newly added ADRs: [ADR-SI-014](../adr/ADR-SI-014.md), [ADR-SI-015](../adr/ADR-SI-015.md), [FIN-003](../adr/FIN-003.md). All PROPOSED.
