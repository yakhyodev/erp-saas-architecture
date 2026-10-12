# MODULE_REGISTRY.md

- **Version:** 0.1 (2026-10-11)
- **Scope:** central cross-module registry; source main and user-provided 14-module audit
- **Governance:** APPROVED user business rules are binding; proposals are not approvals; missing provenance remains unresolved.
- **Source:** [Master Architecture](https://github.com/yakhyodev/erp-saas-architecture/blob/main/docs/modules/MASTER_ARCHITECTURE.md)

## Logical domain ownership — proposed map

> Chat owners below are reconciliation contacts, **not** approved exclusive data ownership. Exact module filenames and cross-module internal claims remain subject to local ADR review.

| Chat / Owner | Domain responsibility | Cross-module contracts | Status |
|---|---|---|---|
| 00 Master Architecture | Overall architecture, governance, global registries | All modules | PROPOSED |
| 01 Monetization & Billing | SaaS subscription, seat/usage rating, wallet/credit ledger | Tenant, Mobile, Sales, integrations | PROPOSED |
| 02 Multi-Tenant | Tenant/company/branch structure and isolation | Billing, Security, DB | PROPOSED |
| 03 Database Architecture | Persistence model, transactional integrity, partition/HA | Backend, Tenant, Infra | PROPOSED |
| 04 Backend Architecture | Domain contracts, application workflows | All business modules | PROPOSED |
| 05 Sales & Inventory | Sales, purchase, catalog, stock movement | Finance, CRM, Billing | PROPOSED |
| 06 Finance & Accounting | Customer ERP GL, AR/AP, payments and posting | Sales, Inventory, CRM | PROPOSED |
| 07 CRM & Distribution | Party/customer relationships, outlets, agents, distribution | Sales, Finance, Mobile | PROPOSED |
| 08 UI/UX Architecture | Workflow UX, accessibility and web interaction | All domains, Localization | PROPOSED |
| 09 Mobile & Offline | Offline operations, synchronization, field use | Billing, API, Security | PROPOSED |
| 10 Security & Permissions | Identity, membership, RBAC/ABAC authorization | Tenant, all modules | PROPOSED |
| 11 Infrastructure & Scaling | Runtime, deployment, observability, DR/HA | DB, Backend | PROPOSED |
| 12 Integrations & SalesDoc API | External APIs, connector contracts and evidence | Security, domain owners | PROPOSED |
| 13 Localization & Alphabet | Translation/locale and alphabet lifecycle | Web, Mobile, Search | PROPOSED |
| 14 Audit / Reconciliation | Cross-module gap audit and provenance tracking | Master, every owner | PROPOSED |

## Proposed disputed ownership contracts (NOT APPROVED)

| Issue | Proposed contract/variants | Pending user/owner decision |
|---|---|---|
| AR-001 | Tenant organizational boundary; BillingAccount payment obligation entity; Wallet ledger owner Billing. Variants: 1 wallet/tenant vs 1 wallet/company vs flexible billing accounts. | Cardinality + who pays for branches/companies |
| AR-002 | Catalog/Product canonical owner: Sales/Inventory shared catalog boundary vs separate Catalog module; Inventory owns quantities/valuation; CRM owns customer-facing assortment relations. | Global vs company catalog + SKU definition |
| AR-005 | Sales owns commercial doc state; Stock owns inventory movements; Finance owns GL posting. Domain facts link via IDs, not mutable shared state. | Lifecycle trigger matrix and atomicity policy |
| AR-008 | Party as cross-domain reference with role projections (Counterparty, Payer, Outlet) vs independent bounded masters with mapping. | Canonical party ID owner and merge/conflict policy |
| AR-009 | Offline device records occurred_at and submitted_at; Billing controls immutable rating and ledger, but occurrence-vs-receipt month undecided. | Month attribution, late event and reversal policy |
| AR-010 | Sales owns source doc lifecycle; Billing owns charge event dedup/rating. Variants: create, confirm, post, finalize as charge boundary. | Billable trigger and credit/refund behavior |
| AR-011 | Integration owner keeps evidence ledger: specified / mocked / sandbox verified / production verified (proposal, not evidence). | Required evidence level per API publication/use |
| AR-012 | UX owns cross-product workflow acceptance; domain owns business invariants. | Definition of Done and scenarios |
| AR-013 | Identity global principal; Tenant membership scoped; module permission enforcement domain-side (proposal). | Membership and permission authority, scopes and delegation |

Unresolved AR issues are tracked in [OPEN_QUESTIONS.md](https://github.com/yakhyodev/erp-saas-architecture/blob/main/OPEN_QUESTIONS.md). Nothing in this map changes other chats' ADR status.


## Round 2 v0.2 — disputed authoritative writers (PROPOSED ONLY)

| Concept | Candidate authoritative writer | Alternative / unresolved conflict | Dependents | Approval |
|---|---|---|---|---|
| Catalog/Product/SKU | **Catalog** module within Backend/Sales boundary, owns canonical Product/SKU identity | Global/shared catalog vs tenant/company catalog, and container (standalone module vs Sales-owned Catalog) remain OPEN | Sales, Inventory, Purchasing, CRM, Integration | PROPOSED / OPEN AR-002 |
| Commercial Pricing | **Pricing** owns price books, effective prices and quote policy; Sales consumes immutable price snapshot | Pricing service inside Sales vs standalone; promotions/contract overrides and customer-tier precedence OPEN | Sales, CRM, Billing (distinct SaaS tariffs), Finance | PROPOSED / OPEN AR-002 |
| Sales | Owns order/sales document commercial lifecycle and billable *source facts*, not price master/stock balance/GL | Confirmation vs shipping trigger and return lifecycle unresolved | Inventory, Finance, Billing | PROPOSED / OPEN AR-005/010 |
| Inventory | Owns stock quantity, reservation, movement and inventory valuation facts | Exact GL recognition trigger and negative-stock/valuation policy OPEN | Sales, Purchasing, Finance | PROPOSED / OPEN AR-005 |
| Purchasing | Candidate separate owner for purchase-order/receipt commercial lifecycle | MVP inclusion/exclusion OPEN; not automatically subsumed under Sales | Inventory, Finance, Catalog | PROPOSED / OPEN MR-18 |
| Party/Counterparty | Candidate canonical **Party** writer in CRM/shared master, with Customer/Supplier role projections | CRM-owned Party vs shared Party module; legal-debtor identity/merge OPEN | Sales, Purchasing, Finance, Integrations | PROPOSED / OPEN AR-008 |
| Payer/ReceivableOwner | Finance owns legal AR debtor attribution; Billing owns SaaS payer/BillingAccount relationship | One Party ID vs mapped identities; third-party payer and wallet legal obligation OPEN | Party, Finance, Billing, Tenant | PROPOSED / OPEN AR-001/008 |
| Outlet | CRM/Distribution owns outlet and territory relation, references Party | Outlet as distinct site vs role of Party/Branch OPEN | Sales, Distribution, Mobile | PROPOSED / OPEN AR-008 |
| Identity/UserCredentials | Security/Identity candidate owns global principal and credential lifecycle | Identity service vs tenant-local identity alternative OPEN | TenantMembership, PermissionEvaluation, Mobile | PROPOSED / OPEN AR-013 |
| TenantMembership | Multi-Tenant candidate authoritative **membership writer** | Existing registry's earlier generic Security-membership ownership conflicts with Security/Multi-Tenant appendices; Security-owned membership alternative OPEN | Roles, auth, tenant switch, seats | PROPOSED / STILL_CONFLICTING AR-013 |
| RoleAssignment | Candidate Security/Permissions policy writer, tenant/branch scoped | Multi-Tenant owner vs Security writer vs split administrative API OPEN | Membership, PermissionEvaluation | PROPOSED / OPEN AR-013 |
| PermissionEvaluation | Security owns shared policy evaluation; every domain enforces operation-level guards | Central-only guards vs domain-side enforcement; authoritative grant/revocation flows OPEN | API, offline sync, reporting, search | PROPOSED / OPEN AR-013 |

### Global dependency graph (PROPOSED, no automatic ownership approval)

```text
Client Web/Mobile -> Identity(UserCredentials) -> TenantMembership -> RoleAssignment/PermissionEvaluation
                               |                                      |
                             Tenant/Company/Branch                    +-> Backend operation guards
                               |                                      +-> Search/Reporting ACL
                               +-> BillingAccount -> Wallet -> SaaS Billing usage/seat/resource ledger
Catalog(Product/SKU) -> Pricing -> Sales docs -> Inventory facts -> Finance ERP GL
          |                        |               |                  ^
          +-> Purchasing (MVP OPEN)  +-> CRM Party/Outlet/Payer ------+
                               |                |
Integration adapters -> domain contracts; Mobile outbox -> authorized backend commands
Domain chargeable facts -> SaaS Billing (not ERP GL); Localization/UX/Audit cover all clients/domains
Reporting (MVP OPEN) <- authorized domain read models
Infra/DB supports all; HA/RLS and production verification remain OPEN
```

**MVP scope OPEN (MR-18):** HR, Manufacturing, Purchasing, Reporting each require explicit include/exclude, ownership and milestone decision. See [Round 2](docs/reviews/ARCHITECTURE_REVIEW_ROUND_2.md), [OPEN_QUESTIONS](OPEN_QUESTIONS.md).


## v0.3 — Current-main authoritative writer/dependency matrix (all candidates unless APPROVED business invariant)

Source: [current-main sync baseline](https://github.com/yakhyodev/erp-saas-architecture/commit/f26d4587dcad20f4bf7e8404e27cdcaeace9a355). This table is the **current coordination proposal** and supersedes older overly broad “Security owns membership” descriptions **as proposal wording only**. One writer per entity is a design constraint; no candidate writer has been unilaterally approved.

| Entity / business fact | Proposed authoritative writer | Readers / dependent domains | Alternative, conflict or status |
|---|---|---|---|
| Tenant, Company, Branch, org scope | Multi-Tenant | Security, Billing, ERP domains | Tenant/Company/LegalEntity mapping OPEN AR-001 |
| LegalEntity relationship | Multi-Tenant organization reference + Finance legal accounting view | Finance, Billing, Sales | Single identity/relationship master and liability snapshot boundary OPEN |
| BillingAccount, Wallet, subscription charge, usage rating | SaaS Billing | Tenant, Security seat input, Sales/Mobile usage source | Company 1:1 account 1:1 wallet **SELECTED_PENDING_APPROVAL**, enterprise shared payer OPEN |
| Global User / IdentityAuth / Credentials / sessions | Security Identity domain | Multi-Tenant membership, Backend, Mobile | Exact service packaging and auth policy PROPOSED |
| TenantMembership state and version | Multi-Tenant **recommended candidate** | Security, Backend, Billing seat attribution | Security umbrella wording alternative OPEN AR-013; never dual writer |
| RoleAssignment and PolicyDefinition | Security/Permissions candidate | Backend enforcement, Tenant scope | Multi-Tenant-owned grants vs split administration alternative OPEN |
| PermissionEvaluation / operation enforcement | Security policy evaluator; Backend & domain guard enforces | All entrypoints, search, offline, reports | Cache/version/revoke freshness OPEN MR-11 |
| Catalog/Product/SKU/UoM | Logical Catalog owner | Sales, Pricing, Inventory, Purchasing, CRM/Integration | Tenant/global namespace and container OPEN AR-002; no Sales all-domain master |
| PriceList / commercial Pricing | Logical Pricing owner | Sales and CRM consumers, Finance snapshots | Independent Pricing vs Sales-contained module OPEN; SaaS Billing tariff is separate |
| SalesOrder / sales document lifecycle | Sales | Inventory, Finance, Billing events, CRM | Chargeable transition / partials / returns OPEN AR-010 |
| StockReservation / StockMovement / StockBalance | Inventory | Sales, Purchasing, Finance GL effect | Dispatch leg once; valuation / count / negative stock policy OPEN |
| PurchaseOrder / purchase intent | Purchasing logical owner (if in MVP) | Inventory receipt, Finance AP, Catalog | MVP inclusion/exclusion OPEN MR-18 |
| Canonical Party / Counterparty | CRM-owned Party vs shared Party context **both proposed** | CRM, Sales, Finance, Integration | Canonical writer/merge/aliases OPEN AR-008 |
| Customer relationship, Outlet, territory and visit | CRM/Distribution | Sales, Mobile, UX | Outlet ≠ legal debtor/payer; Party cardinality OPEN |
| LegalDebtor / ReceivableOwner / AR obligation / ERP GL | Finance | CRM Party, Sales, Inventory | Legal snapshot & tax recognition OPEN; FIN-001 provenance UNVERIFIED |
| Payment remitter / third-party payer source | Finance records ERP payment settlement, Party master resolves identity | CRM, Finance | Third-party payer authorization and cross-company allocation OPEN |
| SaaS payer / wallet funder identity mapping | Billing (references verified Party/legal entity) | MT, Finance organizational mapping | Payer ≠ ERP debtor, actual liability mapping OPEN AR-001/008 |
| External IDs (CS_id, SD_id, code_1C), provider evidence | Integrations adapter mapping registry | Domain authoritative import gateways | External IDs never canonical Product/Party/document/wallet identity; LIVE_VERIFIED missing |
| Offline outbox / local operation state | Mobile; Backend owns server accepted command/result | Security, Sales, Billing, CRM | Server reauthorizes; delayed billing month OPEN |
| Translation keys, locales, fallback/aliases | Localization | Web, Mobile, Search, UX | Hidden alphabet OFF; normalization never mutates IDs/amounts |
| Reporting projections / global search | Domain-authorized read models; search index owner TBD | Finance, Inventory, CRM, UX, Security | MVP Reporting decision OPEN; permission-sensitive counts/facets tests NOT RUN |
| Audit/observability, deployment/HA | Platform/Infrastructure, owner-of-source business facts | All | Initial HA/SLO, DR and test evidence OPEN |

**Dependency contracts:** Sales commercial confirm ≠ stock movement ≠ Finance GL post ≠ SaaS charge. Database §18 selects owner-specific ACID+outbox as **preferred PROPOSED**, with older §6 one-transaction option retained as historical non-selected proposal; Finance accounting triggers remain OPEN. Integration is adapter, not Product/Party/GL/Wallet writer. Backend cannot bypass current membership and grants. Source of reconciliation: [Sales ADR-SI-014](docs/adr/ADR-SI-014.md), [ADR-SI-015](docs/adr/ADR-SI-015.md), [Finance FIN-003](docs/adr/FIN-003.md), [Billing §21](docs/modules/BILLING_ARCHITECTURE.md), [DB §18](docs/modules/DATABASE_ARCHITECTURE.md), [MT §20](docs/modules/MULTI_TENANT_ARCHITECTURE.md), [Security §21](docs/modules/SECURITY_ARCHITECTURE.md).

**MVP OPEN:** HR / Manufacturing / Purchasing / Reporting. No scope selection implied.

---

## v0.4 — User-approved MVP Billing cardinality and proposed authority (2026-10-12)

| Business object | Candidate writer/validator | User decision vs technical status |
|---|---|---|
| Company/Branch/LegalEntity organizational link | Multi-Tenant; Finance validates legal evidence | **APPROVED_BY_USER:** Company has distinct LegalEntity, Branch no separate wallet; registration contracts PROPOSED |
| BillingAccount and Wallet/immutable SaaS ledger | Billing | **APPROVED_BY_USER:** one distinct BillingAccount + Wallet per MVP Company; ledger mechanism PROPOSED |
| SaaS legal liability and third-party funder | Billing contract snapshots and payment evidence; Finance legal verification | **APPROVED_BY_USER:** Company LegalEntity remains debtor even with authorized third-party top-up; verification/refund policy OPEN |
| Billable Seat economic identity | Billing, consuming authoritative membership state | **APPROVED_BY_USER:** user × Company, no Branch/device duplicate; state/proration OPEN |
| Shared Resource Usage responsibility assignment | Billing financial writer; Multi-Tenant validates Company; Security enforces permissions | **APPROVED_BY_USER:** one designated Company bears Tenant-shared measured usage; assignment dual-role workflow PROPOSED |
| Metered usage event | Source resource owner; Billing rates and charges | Company/Branch usage to source Company APPROVED_BY_USER; API and metering units PROPOSED |

[ADR-BILL-011](docs/adr/ADR-BILL-011.md) is the canonical scope/provenance. SaaS wallet writer is never ERP Finance, while FIN-003 Party writer choice remains OPEN. Older v0.3 company-cardinality uncertainty is historical for the new MVP scope. No change to identity/membership technical ADR sign-off or Codex readiness.

---

## v0.5 — User×Company Seat economic occurrence ownership (2026-10-12)

| Responsibility | Candidate authoritative writer | Status |
|---|---|---|
| Principal/UserCredentials | Identity/Auth | PROPOSED architecture |
| TenantMembership/version/company+branch scopes | Multi-Tenant | PROPOSED pending AR-013 technical sign-off; never dual writer |
| Role/grant/revoke evaluation | Security; Backend/domain enforces | PROPOSED; UI hide is not authorization |
| Seat billable snapshot, membership-state rating, month/day accrual and SaaS Wallet posting | Billing | PROPOSED writer; **APPROVED_BY_USER only Seat user×Company scope**, not economic cutoffs |
| Company LegalEntity/BillingAccount/Wallet liability | MT legal org reference + Billing contract ledger | User-approved MVP Company 1:1:1:1, detailed technical mapping PROPOSED |

Source [ADR-BILL-012](docs/adr/ADR-BILL-012.md) and [ADR-BILL-011](docs/adr/ADR-BILL-011.md). Economic dedup should key by canonical user×Company, not number of membership/branch/device records. No change to FIN-001 provenance or original permission owner technical approval.
