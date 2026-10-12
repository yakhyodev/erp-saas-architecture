# ADR-010 — Delegated MVP architecture baseline and cross-domain owner contract

- **Status:** `PROPOSED` — `DESIGN_FROZEN_UNDER_USER_DELEGATION` (design defaults selected; legal/commercial unknowns not invented)
- **Date:** 2026-10-12; **Owner:** 00 Master Architecture
- **Source authorization:** user in this Project explicitly asked assistant to choose best remaining architecture approaches independently and prepare for Codex, without further synchronous questions. This authorizes architecture selection and drafting, **NOT** unverified country-specific fiscal contracts, fictitious pricing, claims of tests passed, or autonomous writing of product code.
- **Evidence:** [Round 3 review](../reviews/ARCHITECTURE_REVIEW_ROUND_3.md), [ADR-BILL-011](ADR-BILL-011.md)–[014](ADR-BILL-014.md), existing 00–13 module documents.

## 1. Problem, alternatives, rationale
Independent multi-company SaaS has contradictory possible domain writers and expensive infrastructure choices; implementation without an explicit baseline risks double charge/stock posting and tenant leakage. Alternatives: microservice-first vs modular monolith; full dedicated DB for every tenant vs shared schema; pooled SaaS wallet vs company-specific; direct cross-domain ACID vs owner-local outbox. **Selected for architecture handoff:** bounded-context modular monolith, multi-tenant shared-schema PostgreSQL guarded by end-to-end access enforcement + conditional FORCE RLS, company wallet business model, owner-local transactions/outbox; extracted services, separate DB and sharding only on evidence. Selection uses low operational cost, fewer network hops and transaction auditability. These are staged technical defaults, not claims of benchmark success.

## 2. Candidate technology and packaging

- **Backend:** Go modular monolith, contract-first versioned REST/OpenAPI (API schema compatibility), domain service interfaces; source-owner write only. One process/deployable initially, clear module boundaries; no mandatory microservices.
- **Persistence:** PostgreSQL as single logical primary, composite `tenant_id` integrity, append-only financial/movement ledgers, required transactional outbox/inbox and idempotency. A transaction only covers a domain owner's durable state/outbox; no expectation of one cross-domain Sales+Inventory+Finance+Billing ACID.
- **Web:** Next.js task-first role dashboard, server-side scoped authorized reads. **Mobile:** Flutter, encrypted local scoped outbox, server accepted vs local queued never confounded. `uz/en/ru` public, hidden future Uzbek alphabet OFF.
- **Workers:** start with bounded Postgres-backed job/outbox polling under separate audited worker identity; queue broker, Redis/cache and search engine are optional only when performance/isolation measurements justify. Never use in-memory cache as financial authority.
- **Operational Stage A (internal MVP pilot only):** primary + encrypted offsite base backups/WAL/PITR + periodic verified isolated restore, **NOT HA**. Stage B primary/standby with fencing before any production service needing committed uptime/RTO guarantee; Stage C measured automated failover. Region, backups retention, numerical RTO/RPO/uptime and hosting budget must be explicitly configured and validated, **not asserted**.
- All technical selections `DESIGN_SELECTED_PROPOSED`, not retroactive approval of legacy ADR-003/DB/BE/INFRA.

## 3. One authoritative writer per aggregate (candidate, ready for domain technical sign-off)
| Domain | Writer | Non-writer consumer / guard |
|---|---|---|
| IdentityAuth, global credentials, sessions | Identity component in Security/Auth | TenantMember refs; no second credential store |
| Tenant, Company, Branch, TenantMembership and version | Multi-Tenant | Security consumes, Billing reads versioned facts; membership revoke flows |
| RoleAssignment, PolicyDefinition, PermissionEvaluation | Security/Permissions | Backend/domain enforce on each operation, cached grants require authoritative freshness |
| Canonical Party/legal counterpart identity | **Shared Party bounded context** in monolith | CRM owns Customer/Outlet/agent relationship; Finance AR debtor immutable snapshot; Integration only external-ID mapping |
| Product/SKU/UoM catalog | **Catalog bounded context**, tenant-scoped ID | Sales, Purchasing, Inventory, CRM read; no duplicate Product master |
| PriceList/pricing offers | Pricing | Sales consumes immutable quote/order price snapshot; CRM assortment no master price |
| SalesOrder/commercial lifecycle | Sales | Inventory physical stock not mutated on confirm |
| StockBalance, movement, reservations and valuation source | Inventory | Dispatch decrements once; invoice does not decrement again; GL derives separate fact |
| PurchaseOrder and GoodsReceipt commercial records | Purchasing | Accepted receipt causes exactly-once Inventory receipt transition |
| ERP GL, AR/AP, period close, legal debtor open items | Finance | Company legal identity linked; ERP ledgers never SaaS Wallet |
| SaaS BillingAccount/Wallet/Seat/Usage/rating/negative period | Billing | actual-only charges, no forecast credit limit; LegalEntity liability per ADR-BILL-011 |
| External IDs, provider compatibility, import mapping | Integrations adapter | External provider not authoritative for internal identity, never direct Wallet writer |
| Offline operation queue | Mobile | Backend/domain owns authoritative accepted command; Security reauthorize at reconnect |
| Translation and search alias contracts | Localization; authorized read-model owner | Search results/counts/facets/recents scoped by Security; no canonical ID mutation |

Cross-module schema API version and event ownership require technical review; no writer may silently take another writer's entity. Party shared context choice is a `DESIGN_SELECTED_UNDER_DELEGATION`, not a verified legacy approval.

## 4. Financial / inventory effect matrix — logical default
| Source transition | Inventory physical on-hand | ERP Finance GL | SaaS billing |
|---|---|---|---|
| Sales draft/submit | None | None | None |
| SalesOrder confirmed | Optional inventory reservation, **not physical decrement** | No default AR/revenue | Candidate once-per-logical-order fee only if published tariff |
| Stock dispatch | Once per unique dispatch leg | Inventory/COGS candidate once upon domain-approved posting policy | No repeated SalesOrder base |
| Sales invoice POSTED | No second stock debit | AR/revenue/tax recognized ONLY by legal jurisdiction-configured policy; otherwise `PENDING_ACCOUNTING_POLICY` | No second SalesOrder base by default |
| Goods receipt POSTED | One accepted Inventory increment | GRNI/AP/valuation under approved accounting policy | GoodsReceipt billable only with configured published fee |
| Transfer COMPLETED | Dispatch/receipt legs handled once by Inventory, not by finalization event | No arbitrary revenue | Candidate once-per-logical-transfer fee with published tariff |
| Return/credit note | Linked reversing movement, never silent delete | Original-linked credit/reversal when legally appropriate | No auto refund; linked adjustment policy |
| Offlined/Imported | No client stock/GL/Billing side-effects | Only authorized normalized final command | Historical imported documents excluded by default |

Finance tax jurisdiction, currency, chart-of-accounts, recognized tax point, GL posting triggers, cost method and legal invoice format are **LEGAL_ACCOUNTING_PROFILE_REQUIRED** before those posting paths can run; schema/guards may be designed while disabled. Choose **moving-weighted-average inventory valuation** and deny negative stock as `PROPOSED` MVP default, pending business/accounting sign-off. Reservation TTL should be configurable, not guessed as a contract. No double effects after inbox replay, original-linked immutable correction.

## 5. Cross-cutting MVP policies selected for design (but not legally approved)
- **Billing:** per-Company separate LegalEntity/Wallet and Seat scope approved directly by user ([BILL-011](ADR-BILL-011.md)). Negative month anniversary-clamp and post-expiry economic action gate [BILL-013](ADR-BILL-013.md) PROPOSED; document triggers/marginal volume tiers and price-effective graph [BILL-014](ADR-BILL-014.md) PROPOSED. Invoice/credit/refund/third-party settlement needs provider/law evidence.
- **Tenant isolation:** shared-schema Postgres design plus tenant/company/branch scoped authoritative checks, tenant composite FKs and conditional FORCE RLS for supported tables; transaction-local context and worker/connection-pool tests mandatory before unrestricted multi-tenant production. Dedicated DB placement only for qualifying enterprise needs and proven migration procedure.
- **Membership:** Multi-Tenant one membership writer; Security roles/grants; Backend/domain security guard; deny-on-stale/unknown on mutations and delayed offline acceptance, no privilege fallback. Revoked device cannot be remotely wiped until next connectivity; local encryption and retention must be policy-bound.
- **Party:** one tenant-scoped canonical Party; no free-text remitter/payment fallback to debtor; CRM Outlet != LegalEntity debtor. Merge uses aliases/audit and immutable financial debtor snapshot. Third-party SaaS funder separate from ERP remitter.
- **MVP scope:** include tenant/company/branch administration, auth/roles, Catalog/Pricing, CRM Party/Customer/Outlet, SalesOrder/dispatch, core Inventory/receipt/transfer, minimal Purchasing, Billing/Wallet, **basic** Finance AR/AP/GL behind legal policy gate, Uzbek/Russian/English UX, mobile agent/order offline under server authorization, and basic read-only reports. **Defer** Manufacturing, Payroll/full HR, advanced BI, production two-way SalesDoc sync and enterprise consolidated wallet until subsequent roadmap. SalesDoc documented API adapter remains read-only and disabled until LIVE_VERIFIED.
- **Privacy:** GPS/media only by lawful purpose/consent and defined retention; no default public/location tracking or implicit GPS upload. Retention days and legal hold jurisdiction must be explicitly established; encrypted device cache/quarantine default design.
- **Experience:** user-approved higher-clarity-than-Sales-Doctor goal; UX-T01–T20 scenarios documented, mobile/desktop/locale/a11y quality gate; no empirical competitive win until paired usability pilot with actual participants.
- **Performance/DR:** define workload envelopes and measurable p95/p99 + storage/worker budgets before numeric SLA promise. Stage A NOT HA, no fake restore/failover benchmark.

## 6. Risks and acceptance
High: SaaS liability routing, unauthorized cross-tenant data access, double-posted GL/stock/Walet, invalid fiscal recognition, backdated invoice changes, third-party payer fraud, offline device invalid rights. Gate on typed economic occurrence & idempotency, source/local outbox, server authorization, RLS/pool negative isolation matrix, staged payment settlement, financial reconciliation, restore drill and actual UX pilot. **These are design acceptance specifications, NOT PASSED tests.**

## 7. Status, dependencies and Codex authorization
This ADR selects **coherent default architecture** under explicit user delegation, but unresolved legal/tax/pricing/SLA facts cannot be manufactured. `DESIGN_FROZEN_UNDER_DELEGATION` means no more preference questions for architecture drafting; it is NOT proof of statutorily compliant finance, measured availability, provider API LIVE_VERIFIED, or formal sign-off of every historical ADR. Codex **handoff materials may be prepared**; current product-code permission remains withheld by project's existing no-code phase and the user asked to prepare for Codex, not to deploy production. Preflight must not replace approvals with guessed values. See [CODEX_START_HERE](../handoff/CODEX_START_HERE.md).
