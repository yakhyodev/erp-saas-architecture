# SalesDoc API V2 — Provider Compatibility & Evidence Appendix

- Sana: 2026-10-12
- Status: **PROPOSED** (ERP integratsiya kontrakti); external API claims **DOCUMENTED**, runtime **NOT_VERIFIED**
- Owner: 12 Integrations & API
- Historical baseline: [016573c8](https://github.com/yakhyodev/erp-saas-architecture/blob/016573c8426ed3ea3b1eacbd0dc81adb67861220/docs/modules/INTEGRATION_ARCHITECTURE.md)
- Current canonical architecture: [INTEGRATION_ARCHITECTURE.md](INTEGRATION_ARCHITECTURE.md)
- Sources: [SalesDoc V2 README](https://github.com/Clever91/salesdoc-api-doc/blob/main/en/README.md), [general](https://github.com/Clever91/salesdoc-api-doc/blob/main/en/01-general.md), [02](https://github.com/Clever91/salesdoc-api-doc/blob/main/en/02-get-references.md), [03](https://github.com/Clever91/salesdoc-api-doc/blob/main/en/03-get-visits-warehouse.md), [04](https://github.com/Clever91/salesdoc-api-doc/blob/main/en/04-get-orders.md), [05](https://github.com/Clever91/salesdoc-api-doc/blob/main/en/05-get-finance.md), [06](https://github.com/Clever91/salesdoc-api-doc/blob/main/en/06-get-extra.md), [07](https://github.com/Clever91/salesdoc-api-doc/blob/main/en/07-set-references.md), [08](https://github.com/Clever91/salesdoc-api-doc/blob/main/en/08-set-warehouse-orders.md), [09](https://github.com/Clever91/salesdoc-api-doc/blob/main/en/09-finance-photo-extra.md), [10](https://github.com/Clever91/salesdoc-api-doc/blob/main/en/10-contragent.md).

## Evidence classification (AR-011)
- **DOCUMENTED**: explicitly specified in publicly available SalesDoc API V2 docs. DOES NOT imply a successful production call.
- **LIVE_VERIFIED**: authenticated, authorized live environment result + redacted transcript, environment/tenant scope, timestamp, reviewer and exact API version; **none exists for this project**.
- **INFERRED**: engineering interpretation of published behavior, test required.
- **UNVERIFIED**: not supported by documentary or runtime evidence; cannot claim compatibility.
- **NOT_VERIFIED** = sandbox / production runtime test status for *every* SalesDoc API behavior in this appendix. Do not interpret DOCUMENTED as LIVE_VERIFIED.

## Historical baseline recovery (R2-DOC-02)
Original 2026-10-11 baseline records source links, 10-domain GET/SET matrix, method dispatch, login, filial, CS_id/SD_id/code_1C, batching/pagination, 400/401/402/403/407/429, special throttles, ordinary/contragent server distinction, staging, incremental reconciliation, single-writer cutover and 1C caveat. Current architecture v0.3 contains part of the catalog, but this appendix is the separate, auditable provider-specific authority for **documented source evidence**. Nothing here supersedes an approved ERP domain contract.

## Provider protocol (each entry: DOCUMENTED; runtime NOT_VERIFIED)
| Claim | Documentation scope | Adapter interpretation (PROPOSED) |
|---|---|---|
| Single JSON POST `/api/v2`, `method` dispatch | en/01-general | Translate to internal versioned domain command; never expose SalesDoc envelope as ERP canonical API |
| `login` returns `userId` + `token`; requests use `auth` | en/01-general | Tenant-scoped encrypted secret ref; serial token renewal; new login may invalidate prior token |
| `filial.filial_id` optional/0 for relevant methods | en/01-general | Explicit connection/source filial → ERP branch binding; scope-check on each import |
| GET `limit` / `page`; documented default 1000 | en/01-general | Method-scoped pagination/cursor; server-configurable limits, overlap + duplicate detection |
| SET `data` batch default cap 250, configured by server | en/01-general, en/07-set-references | Split batches; treat 407 as too large per documented envelope |
| `period.from`, `period.to`, `period.by=create|update` where supported | en/01-general | Capture actual timezone, cutoff, overlap, opaque stable source IDs |
| `filter.include=sd|1c|all` only method-dependent | en/01-general | Capability map per method; no blanket filter assumption |
| `CS_id`, `SD_id`, `code_1C`; SET lookup priority CS_id > SD_id > code_1C | en/01-general, en/07-set-references | Mapping scoped by tenant/provider/connection/server/filial/entity; never assume globally unique |
| `status:false`, partial results, `completed`/`error`, `error.data` | en/01-general, en/07-set-references | HTTP success does not mean full business success; item-level quarantine |

## Capabilities (each named method **DOCUMENTED**, runtime **NOT_VERIFIED**)
| Published section | Documented methods / feature examples | Suggested ERP consumer/owner (PROPOSED, not authoritative reassignment) | Production proof |
|---|---|---|---|
| 02 — References | `getValyutaType`, `getUnit`, `getClientCategory`, `getProduct`, `getProductCategory`, `getProductSubCategory`, `getProductGroup`, `getPriceType`, `getPrice`, `getClient`, `getAgent`, `getExpeditor`, `getSupervisor`, `getPaymentType`, `getPayment`, `getTerritory`, `getBrand`, `getTradeDirection`, `getClientChannel`, `getClientType` | Catalog / Pricing / CRM / Finance | NOT_VERIFIED |
| 03 — Visits & warehouse | `getVisit`, `getWarehouse`, `getDefectWarehouse`, `getStock`, `getStockForDate` | CRM/Distribution, Inventory | NOT_VERIFIED |
| 04 — Orders | `getOrder`, `getOrderDefect`, `getOrderReplace` | Sales, Returns | NOT_VERIFIED |
| 05 — Finance | `getBalance`, `getConsumption`, `getCashbox` | Finance GL/AR/AP; not SaaS Billing wallet | NOT_VERIFIED |
| 06 — Extra | `getInventory`, `getShipper`, `getPaymentConfirmation`, `getBonus`, `getPRCategory`, `getPhotoReport`, `getFilials`, `getStoreLog`, `getClientPending`, `getCorrection`, `getPurchase`, `getMovement`, `getVsExchange`, `getMovementBetweenFilial`, `getTag`, `getMovementFilialStatus`, `getAgentLastTime`, `getSubstatus`, `getSubstatusLog` | Purchasing, CRM, Inventory, Tenant, Audit | NOT_VERIFIED |
| 07 — SET references | `setValyutaType`, `setUnit`, `setClientCategory`, `setClientChannel`, `setClientType`, `setProduct`, `setProductCategory`, `setProductSubCategory`, `setProductGroup`, `setPaymentType`, `setPriceType`, `setPrice`, `setAgent`, `setExpeditor`, `setClient`, `setClientPhone`, `setTerritory`, `setBrand`, `setVisit` | Provider outbound capability only, disabled until separately approved | NOT_VERIFIED |
| 08 — Warehouse SET | `setWarehouse`, `setStock`, `setPurchase`, `setMovement`, `setVsExchange`, `setMovingToFilial`, `setMovementBetweenFilial`, `setMovementFilialPending`, `setSupplierReturn`, `setExcretion`, `setCorrection` | Inventory/Purchasing; no direct ledger mutation via Integration | NOT_VERIFIED |
| 08 — Orders SET | `setOrder`, `setDeletedOrder`, `setStatus`, `setCode`, `setOrderDefect`, `syncOrder`, `setSubstatus` | Sales only through owner-reviewed export contract | NOT_VERIFIED |
| 09 — Finance/photos/extras | `setPayment`, `setBalance`, `setCurrentBalance`, `setConsumption`, `setPhoto`, `getPhoto`, `getFilials`, `getStoreLog`, `getClientPending`, `deleteClientPending` | Finance/Media/CRM; writes guarded and permissioned | NOT_VERIFIED |
| 10 — Contragent-server only | `getContragent`, `setContragent`, `setClient.contragent`; `getPayment` includes payer + optional client; `getBalance` payer-oriented | CRM Party/Outlet and Finance debtor attribution; AR-008 OPEN | NOT_VERIFIED |

Availability is method-dependent and server-mode-dependent; this is **not** proof all features work in every deployment. Method catalog is not a Sales Doctor UI feature list.

## Documented error / request policy (DOCUMENTED; runtime NOT_VERIFIED)
| Signal | Meaning in docs | PROPOSED safe integration treatment |
|---|---|---|
| 400/402 | request validation/data or method issues | validate schema, quarantine; do not blindly retry |
| 401 | auth/token rejected | serialized credential renewal, only safe retry |
| 403 | permission denied | block connection/job pending permission review |
| 407 | batch/request size restriction | lower batch size; record provider observed cap |
| 429 | throttled | bounded backoff/jitter per connection and method |
| partial SET success | some items processed, others failed | item-scoped reconciliation; do not re-send already accepted writes |
| timeout on write | final effect unknown (INFERRED) | state UNKNOWN, read-back/reconcile before retriggering side effects |

**Rate limits (DOCUMENTED; runtime NOT_VERIFIED):** `setStock` 5 calls/5 seconds; `setCurrentBalance` 3 calls/5 minutes; `getStockForDate` 1 call/5 minutes; `getStoreLog` 5 calls/5 seconds, server-local window 20:00–07:00. Other methods: unspecified / UNVERIFIED. No invented global request ceiling.

## Source-specific identity and payer semantics
- **DOCUMENTED, NOT_VERIFIED:** `CS_id` source-side custom/branch-sensitive ID; `SD_id` SalesDoc ID; `code_1C` 1C external code. Keep all three with namespace, revision and mapping provenance.
- **DOCUMENTED, NOT_VERIFIED:** ordinary server associates balance with client; contragent server separates legal payer (contragent) and outlet (client). Contragents may exist without outlets; payments may link directly to payer and null outlet. SalesDoc may associate orders with contragent via outlet.
- **PROPOSED, AR-008 OPEN:** external client → candidate Outlet/CustomerRole; external contragent → candidate Party/PayerRole; Finance must bind legal ReceivableOwner in own immutable ledger snapshot. Party authoritative writer CRM vs shared Party remains unresolved; product canonical owner/scope AR-002 remains unresolved. Integration owns ONLY external mapping and conflict quarantine.
- **UNVERIFIED:** complete merge/split history, enterprise payer eligibility, exhaustive deletion feed and 1C end-to-end two-way correctness.

## Migration contract (PROPOSED; no live tests)
1. SOURCE INVENTORY: permission, server mode, source branches, method-specific evidence.
2. READ-ONLY EXTRACTION: bounded period windows/pages, fingerprints, raw payload provenance, source timezone; duplicate/skip detection.
3. MASTER MAPPING: Unit/Product/SKU and Customer/Party/Outlet/Payer; collisions resolved with domain owner approval.
4. TRANSACTION STAGING: orders, returns, purchases, movements, payments, balances as observations, not automatic GL or StockMovement.
5. RECONCILIATION: documents and revisions, stock by branch/warehouse/as-of cutoff, money by payer/currency, exceptions and signatures.
6. PARALLEL OBSERVATION: one writer per process; delta overlap, snapshot and handover watermarks.
7. CUTOVER: authorized freeze and final catch-up, owner sign-offs, immutable evidence bundle; rollback plan before any switch.
8. POST-CUTOVER: monitor divergence, no dual-active writers, controlled corrections and approved credential retirement.

**Historical imports and charge eligibility:** source archival load is NOT an automatic new billable ERP document. Stable logical external document key + source revision + canonical doc + business occurrence correlation; same-key/different-payload is a CONFLICT/revision, not duplicate economic charge. Billing alone rates an approved chargeable lifecycle fact; Sales owns sale document state. Chargeable type, transition, imported historical eligibility and reversals are OPEN AR-010.

## Integration safety matrix — PROPOSED, not source capability assertions
| Mode/operation | Writer | Failure / idempotency | Allowed gate |
|---|---|---|---|
| Read-only source pull | SalesDoc source | Retry GET with overlapping windows and fingerprint dedup | Documented method + authorized credentials |
| Staging and mapping | Integration | Immutable capture; conflict on same ID different payload; quarantine | Tenant/company/branch source validation |
| Sales command import | Sales domain | stable command ID and occurrence ID, owner-side outbox | Sales transition policy and mapping approved |
| Inventory posting | Inventory domain | unique movement leg and append-only reversal | Inventory owner sign-off; no import snapshot auto-post |
| Finance posting | Finance domain | unique GL/AR occurrence and compensating adjustment | Finance owner sign-off |
| Billing charge | Billing | at-least-once fact delivery, once-only economic posting keyed by approved chargeable occurrence | AR-010 type/transition/import/period policy APPROVED |
| SalesDoc SET export | one designated source writer per process | ambiguous timeout → UNKNOWN/reconcile; no blind SET replay | verified permission, live evidence, explicit opt-in |
| Webhook (future) | external sender; ERP validates | verify signature or approved alternative, anti-replay timestamp/nonce, tenant binding | UNVERIFIED support; never assume native SalesDoc webhook |
| Cutover/rollback | Cutover coordinator + domain owners | frozen source checkpoint, high-watermark, mapped counts, money and inventory reconciliation | signed acceptance, reversal policy, test evidence |

## Versioned contracts and cutover evidence (MR-10 / MR-19)
- **PROPOSED** `IntegrationImportCommand.v1`: tenant_id, connection_id, source_entity_kind, source_id_namespace/value, revision/hash, observed_at, source_event_at?, canonical_target_type, mapping_version, command_id, occurrence_id, correlation_id.
- **PROPOSED** `IntegrationObservation.v1`: distinct event_id (transport), stable business_occurrence_id only when resolved by domain owner, entity_version, schema_version, tenant/branch scope, occurred_at/received_at, payload_hash, source_ref.
- Version change: additive backward-compatible optional fields within minor version; breaking changes in major version, compatibility window OPEN. Source adapter cannot unilaterally set Finance posting/Billing charge semantics.
- **PROPOSED cutover evidence bundle:** source snapshot reference; source server mode; list of methods and evidence labels; extracted page counts/checksums; external→canonical mapping review; document/stock/debt reconciliation with cutoff/timezone/currency; exception register; identified writer authority before/after; rollback/compensation plan; sign-off from Sales/Inventory/Finance/Billing/Security; dry-run result **NOT_VERIFIED** pending authorized execution.
- Rollback: before domain posting discard staging with audit; after posting use owner-approved compensating events; after cutover prefer controlled forward repair over risky dual-write rollback. No proven zero downtime, complete history or production recovery RTO.

## Cross-module conflicts / OPEN
- AR-008: CRM Party vs shared Party master; legal payer/ReceivableOwner resolution by Finance. No ownership takeover.
- AR-002: Product/SKU master and catalog scope remain owner decisions.
- AR-010: Billing/Sales determine billable source occurrence, historical import treatment, corrections/reversal and time attribution.
- AR-011: exact production evidence requirements and live access approval unresolved.
- MR-10: versioned schema registry, consumer compatibility/deprecation period, outbox transactional boundary require Backend concurrence.
- MR-19: provider production/sandbox capabilities, deletion behavior, cutover rehearsals and rollback evidence **NOT_VERIFIED**.
- No modifications here to approved Hybrid Prepaid, 1 credit=10 so'm, real negative-balance one calendar month/no estimate, progressive discounts, company/branch platform or supported languages.

## Codex architectural acceptance (NO CODE)
Design read-only adapter contracts and evidence manifest; plan integration-scoped mapping and quarantines; specify owner-issued occurrence keys, replay-safe outbox/inbox, error handling and safe cutover. Product implementation must remain blocked until domain sign-off and required runtime evidence. Do not connect credentials or perform write API calls in this architecture phase.
