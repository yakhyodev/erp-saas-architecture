# INTEGRATION_ARCHITECTURE.md

- Version: 0.1 — 2026-10-11
- Status: **PROPOSED** except explicitly identified approved business decisions
- Scope: SalesDoc API V2, 1C mapping, migration, external adapters
- Evidence: public documentation only; no live credential, server login or external write API used.

## Purpose
Build a self-contained ERP canonical domain model independent of any external provider. The SalesDoc API is a migration/integration evidence source, NOT evidence of every Sales Doctor UI feature. Integration owns transport, mapping, staging and sync, while CRM, Sales, Inventory and Finance own business invariants.

## Source references
[README](https://github.com/Clever91/salesdoc-api-doc/blob/main/en/README.md) · [01-general](https://github.com/Clever91/salesdoc-api-doc/blob/main/en/01-general.md) · [02-get-references](https://github.com/Clever91/salesdoc-api-doc/blob/main/en/02-get-references.md) · [03-get-visits-warehouse](https://github.com/Clever91/salesdoc-api-doc/blob/main/en/03-get-visits-warehouse.md) · [04-get-orders](https://github.com/Clever91/salesdoc-api-doc/blob/main/en/04-get-orders.md) · [05-get-finance](https://github.com/Clever91/salesdoc-api-doc/blob/main/en/05-get-finance.md) · [06-get-extra](https://github.com/Clever91/salesdoc-api-doc/blob/main/en/06-get-extra.md) · [07-set-references](https://github.com/Clever91/salesdoc-api-doc/blob/main/en/07-set-references.md) · [08-set-warehouse-orders](https://github.com/Clever91/salesdoc-api-doc/blob/main/en/08-set-warehouse-orders.md) · [09-finance-photo-extra](https://github.com/Clever91/salesdoc-api-doc/blob/main/en/09-finance-photo-extra.md) · [10-contragent](https://github.com/Clever91/salesdoc-api-doc/blob/main/en/10-contragent.md).

**Evidence classification:** VERIFIED_DOC = stated in API documentation; INFERRED = reasoned; UNVERIFIED = server/UI not confirmed; PROPOSED = ERP design, not approved.

## Documented SalesDoc API V2 protocol — VERIFIED_DOC
- Requests: JSON body via `POST /api/v2`, operation selected by `method` string; not a recommended ERP native public API.
- Authentication: `login` yields `userId` and `token`; next requests use `auth`. Another login may invalidate prior token, requiring serialized connector login.
- Branch: `filial.filial_id` optional/0 for appropriate calls; external branch and database IDs are not internal ERP tenant IDs.
- GET pagination: `page`, `limit`; documented default limit 1000, server-configurable cap. SET batches: documented default max 250 per data group, configurable; exceeding limit may return 407.
- Period: `period.from`, `period.to`, `period.by=create|update` where supported. `filter.include=sd|1c|all` only for specific methods.
- External IDs: `CS_id` (possibly branch-prefixed), `SD_id` (server ID), `code_1C` (1C XML_ID). SET lookup priority CS_id → SD_id → code_1C; conflicting IDs can create duplicates.
- Parse `status:false`, per-item failures and application error codes 400, 401, 402, 403, 407, 429; HTTP success alone does not indicate complete success.
- Specific method throttles: `setStock` 5 per 5 sec; `setCurrentBalance` 3 per 5 min; `getStockForDate` 1 per 5 min; `getStoreLog` 5 per 5 sec, allowed source-server time 20:00–07:00. Do not invent a global limit.
- Ordinary server: client owns receivable. Contragent server: sale point sells while contragent can own debt/payment; `getContragent` and `setContragent` are mode-specific.

## API capability / gap matrix
| Domain | Documented GET examples | SET examples | ERP owner | Gaps |
|---|---|---|---|---|
| Catalog | getProduct, getProductCategory, getBrand, getUnit | setProduct + references | Catalog | SKU uniqueness/variants UNVERIFIED |
| Pricing | getPriceType, getPrice | setPriceType, setPrice | Pricing | discount workflow UNVERIFIED |
| CRM | getClient, getClientCategory, getContragent* | setClient, setClientPhone, setContragent* | CRM | payer versus sale point; *mode-specific |
| Distribution | getAgent, getExpeditor, getTerritory, getVisit | setAgent, setTerritory, setVisit | CRM/Distribution | route optimization UNVERIFIED |
| Sales | getOrder, getOrderDefect, getOrderReplace, getSubstatus | setOrder, setDeletedOrder, setStatus, syncOrder | Sales | status mapping and corrections |
| Inventory | getWarehouse, getStock, getPurchase, getMovement, getCorrection | setWarehouse, setStock, setPurchase, setMovement, setCorrection | Inventory | valuation and posted ledger semantics |
| Finance | getPayment, getBalance, getConsumption, getCashbox | setPayment, setBalance, setCurrentBalance, setConsumption | Finance | double-entry behavior UNVERIFIED |
| Branch | getFilials, getMovementFilialStatus | branch transfer operations | Multi-Tenant/Inventory | external ID namespaces |
| Media | getPhoto, getPhotoReport, getStoreLog | setPhoto | Media | retention rules UNVERIFIED |
| Supplier | getShipper | setSupplierReturn | Purchasing | purchase policy UNVERIFIED |

1C codes, `setCode` or `syncOrder` do not establish a complete two-way 1C integration.

## ERP replacement decisions — PROPOSED
1. Isolate SalesDoc's single-endpoint and method dispatch within an anti-corruption adapter; design versioned ERP domain APIs separately.
2. Use immutable internal ERP IDs; store CS_id, SD_id and code_1C in scoped external identity mappings, not primary keys.
3. Differentiate payer and sale point per business transaction, pending CRM and Finance approval; do not copy SalesDoc server mode wholesale.
4. Keep receivables/payables (Finance) distinct from SaaS prepaid usage balance (Billing).
5. Never automatically post stock snapshots as inventory ledger transactions: require owning-domain reconciliation and opening-balance approval.
6. External API tokens are connector secrets and never ERP end-user credentials.

## Provider-neutral adapter architecture — PROPOSED
External Provider → Transport/Auth Adapter → Provider Parser → Anti-Corruption Normalization → Validation/Staging → Identity Resolution → Owning Domain Import Service → Outbox/Event → Reconciliation.

- **IntegrationConnection:** tenant_id, provider, source server mode, endpoint allowlist, secret reference, permissions, external branch mapping, status.
- **Connector:** serialized login, request envelope, timeouts, error classification, rate-control per method; outbound SET disabled by default.
- **SyncOrchestrator:** durable per tenant/provider/branch jobs; cursors, checkpoints, overlap windows, retry budgets, dead-letter.
- **ExternalIdentityMap:** scoped by tenant, provider, connection, server, filial, entity type, ID kind and value. Conflicts quarantined; never silent merge.
- **Staging:** raw payload reference, source timestamp/hash, schema version, provenance, validation outcome and retention.
- **Reconciliation:** counts, documents/totals/status, stock by warehouse/date, customer debt by payer and currency, pending exceptions.
- **Security:** encrypted secrets, service principals, RBAC, SSRF-safe endpoint allowlist, egress restrictions, audit and PII-redacted logs.
- **Scaling:** compatible with modular monolith and in-process workers initially; durable queue/outbox abstraction allows later separation. Deployment decision belongs Backend/Infra.

## Seven-stage migration — PROPOSED
0. **Permission and source inventory:** source owner consent, methods/branches/mode, accounting scope; no live access at architecture stage.
1. **Read-only discovery:** paginated source references and filial details, capture raw payload and provenance, identify absent deletion feeds.
2. **Master data:** units/categories/products/brands → prices → agents/territories → counterparties/sale points → warehouses, with code_1C collision review.
3. **Historical and open transactions:** orders, returns, purchases, transfers, payments and balances staged, not posted directly; distinguish historical facts and open documents.
4. **Reconciliation:** document totals/status, stock snapshot vs movements, debt/payment per payer/currency/date; audited opening balances if movement history missing.
5. **Optional parallel run:** bounded overlap-period incremental pulls and dedup; explicit single writer per process; no uncontrolled bidirectional writes.
6. **Cutover:** freeze writer, final delta and watermarks, reconcile, financial sign-off, switch authority, retain IDs, documented rollback avoiding double postings.
7. **Post-cutover:** exceptions, read-only archive, privacy retention and approved credential revocation.

## Error control — PROPOSED
401 → serialized token renewal, one safe retry; 403 → permission blocker; 400/402 → validation/manual review; 407 → reduce batch; 429/network → bounded exponential backoff+jitter and per-method throttle. Handle item-level partial SET outcomes and ambiguous write timeout without blind financial or inventory retry. Respect `getStoreLog` server-local overnight window. Page mutation may duplicate/skip rows; use overlapping windows, dedup and reconciliation.

## Cross-module effects
CRM: payer/sale point semantics; Sales: order lifecycle; Inventory: movements/valuation; Finance: debt/opening balances; Multi-Tenant and Database: isolation and ID uniqueness; Security: credentials/retention; Backend/Infra: worker reliability; UX: sync status/cutover review; Billing: external usage metering if approved, distinct from Finance balances.

**Approved business rules unchanged:** Hybrid Prepaid SaaS; Seat + Business Usage + Resource Usage; 1 credit = 10 so'm; one calendar month real Seat/Usage debt without a forecast credit limit; unified company/branch platform; uz/en/ru plus hidden future Uzbek alphabet. Go, PostgreSQL, Next.js and Flutter are recommended, not approved.

## ADR references — ALL PROPOSED
- [ADR-INT-001](../adr/ADR-INT-001.md) — Anti-Corruption Adapter Layer
- [ADR-INT-002](../adr/ADR-INT-002.md) — Scoped External Identity Registry
- [ADR-INT-003](../adr/ADR-INT-003.md) — Read-first Staged Migration
- [ADR-INT-004](../adr/ADR-INT-004.md) — Single Writer and Guarded Outbound Sync
- [ADR-INT-005](../adr/ADR-INT-005.md) — Contragent/SalePoint Normalization

## Open questions
Source change/deletion feed reliability; server timezone; order state codes; currency/tax/valuation reconciliation; permission for outbound writes; per-branch authentication; personal-data retention; cutover SLA; native webhooks not verified; 1C configuration/version.

## Shared-doc synchronization proposals — NOT APPLIED
- DECISIONS.md: register ADR-INT-001..005 as PROPOSED.
- MASTER_ARCHITECTURE.md: link Integration Anti-Corruption Layer, external identity registry and domain ownership.
- REQUIREMENTS.md: audit, staged imports, tenant isolation, reconciliation, read-only default.
- MODULE_REGISTRY.md: Integration relationships with CRM, Sales, Inventory, Finance, Billing, Security, Backend.
- OPEN_QUESTIONS.md: carry unresolved issues above.
- CHANGELOG.md: 2026-10-11 initial proposed Integration Architecture.

## Codex integration specification — DO NOT WRITE CODE YET
Specify provider-neutral connector contract, SalesDoc V2 adapter and method capability catalog, source-mode treatment, tenant-scoped secrets and filial binding, CS_id/SD_id/code_1C conflict resolution, durable job states (queued/running/retry/blocked/reconciled/completed), read-only ingestion/staging, versioned mapping, canonical domain commands, reconciliation reports and explicit cutover gates. Architecture acceptance scenarios: duplicate IDs and filials, both source modes, 401/403/407/429, partial errors, mutated pagination, crash/replay, ambiguous timeout, missing deletion feed, historical currency and date boundaries. No live credentials or outbound API, no unsupported UI assumption, no implementation until ADR review.
