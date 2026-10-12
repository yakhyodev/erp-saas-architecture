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
