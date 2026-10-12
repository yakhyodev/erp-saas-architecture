# INTEGRATION_ARCHITECTURE.md

- Version: 0.2 — 2026-10-11
- Status: **PROPOSED** except explicitly identified approved business rules
- Scope: External integrations, SalesDoc compatibility, migration contracts, API/event evidence.

## Evidence taxonomy (AR-011) — PROPOSED

External integration claims MUST use one of these labels:

| Level | Meaning |
|---|---|
| DOCUMENTED | Exists in published API/schema/documentation. Does not prove production behavior. |
| LIVE_VERIFIED | Verified against an authorized live environment with recorded evidence. Not achieved for SalesDoc in this architecture phase. |
| INFERRED | Reasoned from documentation or domain behavior; requires validation before contract use. |
| UNVERIFIED | Unknown; must not be implemented as a guaranteed capability. |

Current SalesDoc analysis remains DOCUMENTED only. No production API, credentials, sandbox, or write operations were used.

## Canonical integration boundary — PROPOSED

External systems connect through a versioned adapter boundary. External contracts are not ERP domain contracts.

Provider Adapter → Normalization → External Identity Mapping → Staging → Domain-owned Import/Command Contract → Reconciliation

Each adapter contract MUST define:
- provider name and contract version;
- supported methods/events;
- evidence level per capability;
- identity mapping rules;
- ownership of imported facts;
- retry/idempotency rules;
- cutover and rollback evidence requirements.

## AR-008 payer/counterparty/outlet identity mapping — PROPOSED

Open question remains: canonical owner model is not approved.

Candidate models:

1. Single Party registry with roles:
   - Party
   - Counterparty role
   - Payer role
   - Outlet/sales point role

2. Bounded context ownership:
   - CRM owns relationship identity;
   - Finance owns payer/debt identity;
   - Sales owns commercial participant references.

Current proposal:
- never derive payer ownership only from SalesDoc server mode;
- preserve external source relationships in mapping records;
- transactions reference explicit roles, not ambiguous client IDs.

Decision requires CRM + Finance + Sales approval.

## AR-010 imported external document charge lifecycle — PROPOSED

Imported documents MUST have separate lifecycle from native ERP documents.

Proposed states:

DISCOVERED → STAGED → VALIDATED → MAPPED → ACCEPTED → POSTED → RECONCILED → ARCHIVED

Rules:
- imported payload hash prevents duplicate processing;
- external ID + provider + tenant + entity type is not sufficient alone if source revisions exist;
- corrections create reconciliation events, not silent overwrite;
- chargeability is unresolved and belongs to Billing + Sales/Inventory.

No decision is made that imported documents are automatically chargeable.

## MR-10 versioned API/event contracts — PROPOSED

Integration contracts require:

- immutable contract version;
- compatibility policy;
- migration path between versions;
- event schema ownership;
- backward compatibility window;
- deprecation record.

Events are preferred for durable synchronization, but no event technology choice is approved.

## Migration cutover and rollback evidence — PROPOSED

Before cutover:
- source snapshot/checkpoint recorded;
- imported counts reconciled;
- financial/inventory owners sign acceptance;
- rollback point documented.

Rollback MUST not create duplicate documents or reverse financial facts without domain approval.

## Approved business rules preserved

No changes to:
- Hybrid Prepaid SaaS;
- Seat + Business Usage + Resource Usage;
- 1 credit = 10 so'm;
- progressive usage discounts and enterprise contracts;
- real Seat/Usage monthly negative balance behavior;
- unified company/branch SaaS;
- uz/en/ru and hidden future Uzbek alphabet option.

## Existing ADR status

ADR-INT-001..005 remain **PROPOSED** only. No approval transition performed.


---

## v0.3 — SalesDoc API V2 katalogi va migratsiya spetsifikatsiyasi (2026-10-12)

**Holat: PROPOSED.** Ushbu appendix avvalgi AR-008/010/011 va MR-10 yechimlarini aniqlashtiradi, ularni APPROVED qilmaydi. Tekshiruv faqat SalesDoc rasmiy \`en/README.md\` va \`en/01-general.md\`–\`en/10-contragent.md\` Markdown hujjatlari ustida o'tkazildi. **LIVE_VERIFIED = yo'q**. Metodlar ro'yxati mavjud API hujjatlariga tegishli, Sales Doctor interfeysining to'liq funksional ro'yxati emas.

### A. Asosiy API kontrakti — DOCUMENTED

- JSON \`POST /api/v2\`, \`method\` nomi bilan dispatch; \`login\`dan keyin \`auth.userId\` va \`auth.token\`. Yangi login oldingi qurilma tokenini bekor qilishi mumkin. Token lifecycle ERP IAM autentifikatsiyasidan mustaqil.
- \`filial.filial_id\` XML_ID; ba'zi reference metodlar filialga bog'liq emas. Biror \`filial_id\`ni ERP company, branch yoki tenant ID bilan teng deb qabul qilish taqiqlanadi.
- GET sahifalanishi: \`limit\` (default 1000, server sozlashi mumkin), \`page\` (default 1). SET \`data\` group hajmi default 250, server sozlashi mumkin; oshsa 407. API natijasidagi \`status\`, \`result.completed\`, \`result.error\`, \`error.data\` har biri tekshiriladi. HTTP 200 = biznes muvaffaqiyati emas.
- \`period.from/to/by\`, \`sort\` bo'yicha inkremental pull faqat metod qo'llaganda; \`filter.include=sd|1c|all\` faqat \`getClientCategory\`, \`getMovement\`, \`getVsExchange\`, \`getOrder\`, \`getConsumption\` metodlarida amal qiladi.
- \`CS_id\`, \`SD_id\`, \`code_1C\` ko'p manbali ID; SET matching prioriteti CS_id → SD_id → code_1C. Har bir mapping tenant, connection, source-server, branch, entity-type, ID-kind scope bilan saqlanadi.
- Hujjatlashtirilgan limitlar: \`setStock\` 5/5s; \`setCurrentBalance\` 3/5min; \`getStockForDate\` 1/5min; \`getStoreLog\` 5/5s, source server vaqti 20:00–07:00. Boshqa universal limit taxmin qilinmaydi.

### B. Funksiya katalogi → ERP almashtirish matritsasi

| API bo'limi / DOCUMENTED misollar | Ichki ERP domeni | Almashtiriladigan model (PROPOSED) | Inkor etilmaydigan cheklov |
|---|---|---|---|
| 02: \`getProduct\`, \`getProductCategory\`, \`getProductSubCategory\`, \`getProductGroup\`, \`getUnit\`, \`getBrand\`; 07: \`setProduct\`, \`setUnit\`, reference SET | Catalog | ERP Product/SKU/UoM/Category master, tashqi identifikatorlar mapping | Catalog owner/scope AR-002 OPEN |
| 02: \`getPrice\`, \`getPriceType\`; 07: \`setPrice\`, \`setPriceType\` | Pricing | price book/version va domain-owned narx siyosati | ERP tarif va Sales price master bir narsa emas |
| 02: \`getClient\`, \`getAgent\`, \`getExpeditor\`, \`getSupervisor\`, \`getTerritory\`, \`getClientChannel\`, \`getClientType\`; 07 SET | CRM/Distribution | Party/Customer/Outlet/agent/territory projection | payer vs outlet AR-008 OPEN |
| 03: \`getVisit\`, \`getWarehouse\`, \`getDefectWarehouse\`, \`getStock\`, \`getStockForDate\`; 07: \`setVisit\` | Distribution/Inventory | visit history va inventory read model | GPS tracking yoki valuation kafolati emas |
| 04: \`getOrder\`, \`getOrderDefect\`, \`getOrderReplace\`; 08: \`setOrder\`, \`setDeletedOrder\`, \`setStatus\`, \`setOrderDefect\`, \`syncOrder\`, \`setSubstatus\` | Sales | versioned commercial doc + owner-driven state machine | lifecycle event va charge trigger AR-010 OPEN |
| 06: \`getInventory\`, \`getPurchase\`, \`getMovement\`, \`getCorrection\`, \`getVsExchange\`, \`getMovementBetweenFilial\`; 08 warehouse SET, receipt, movement, correction, write-off, supplier return | Purchasing/Inventory | StockMovement, receipt, transfer, reservation alohida | snapshotni ikki marta stockga qo'shmaslik |
| 02/05: \`getPayment\`, \`getBalance\`, \`getConsumption\`, \`getCashbox\`; 09: \`setPayment\`, \`setBalance\`, \`setCurrentBalance\`, \`setConsumption\` | Finance | GL/AR/AP, import opening balance only after owner sign-off | SaaS Billing wallet bilan aralashtirilmaydi |
| 06/09: \`getFilials\`, \`getStoreLog\`, \`getClientPending\`, \`getSubstatusLog\`, \`getAgentLastTime\` | Tenant, CRM, Audit | scope-aware references, external audit observations | full historical change feed UNVERIFIED |
| 09: \`getPhoto\`, \`setPhoto\`, \`getPhotoReport\` | Media | authorized media ingest and retention | complete media history UNVERIFIED |
| 10: \`getContragent\`, \`setContragent\`; \`getPayment\` returning \`contragent\` and \`client\` | CRM + Finance | distinct Party/Counterparty, Payer/ReceivableOwner, Outlet | contragent-server only; payer ownership OPEN |

**1C:** \`code_1C\`, \`setCode\`, \`syncOrder\` hujjatlashtirilgan; bu to'liq avtomatik ikki tomonlama 1C sinxronizatsiya, webhooks yoki barcha 1C konfiguratsiyalarini isbotlamaydi. Bunday imkoniyatlar UNVERIFIED.

### C. Anti-Corruption API Adapter (PROPOSED)

**Flow:** Auth/Endpoint Allowlist → Provider-specific GET/SET transport → Rate-limit scheduler → schema adapter (versioned) → immutable raw/staging → external ID resolution → domain validation/import command → owner transaction + outbox → inbox/reconciliation → audit/monitoring.

**IntegrationConnection contract:** \`connection_id\`, \`tenant_id\`, \`company_scope\`, \`provider\`, \`provider_api_version\`, \`server_mode(observed|unknown)\`, \`endpoint_allowlist_ref\`, \`secret_ref\`, \`branch_mapping_version\`, \`capabilities/evidence\`, \`enabled_operations\`, \`status\`. Credentials are not logged; no cross-tenant credential reuse. Separate source connection ID from domain IDs.

**SyncRun contract:** \`run_id\`, \`connection_id\`, \`tenant_id\`, \`source_entity\`, \`source_window\`, \`cursor/page\`, \`observed_at\`, \`high_watermark\`, \`schema_version\`, \`state\`, \`attempt_count\`, \`reconciliation_id\`. Retries must tolerate mutable offset pages; use overlap + content fingerprint + reconciliation instead of assuming ordered change feed.

**ExternalIdentityMap contract:** \`tenant_id\`, \`provider\`, \`connection_id\`, \`source_server\`, \`filial\`, \`entity_kind\`, \`id_kind\`, \`id_value\`, \`erp_entity_id\`, \`mapping_version\`, \`provenance\`, \`effective_from/to\`, \`conflict_status\`. Same scoped source ID mapping to different ERP IDs → quarantine; different source ID namespaces identifying same subject require audited merge review.

**ImportCommandV1 (proposed wire-level fields):** \`command_id\`, \`schema_version\`, \`tenant_id\`, \`source_connection_id\`, \`source_entity_id\`, \`source_revision_or_fingerprint\`, \`source_occurred_at?\`, \`imported_at\`, \`actor/service_principal\`, \`business_entity_kind\`, \`canonical_payload\`, \`mapping_version\`, \`dry_run\`, \`correlation_id\`. Domain writer validates tenant, permissions, units, referential integrity and allowed lifecycle transition. No direct writes into other owners' tables.

**Canonical event candidate:** \`event_id\` (transport uniqueness), \`business_occurrence_id\` (stable economic occurrence; not same as event ID), \`aggregate_id\`, \`aggregate_version\`, \`event_type\`, \`schema_version\`, \`tenant_id\`, \`company_id?\`, \`branch_id?\`, \`effective_business_at\`, \`recorded_at\`, \`origin\`, \`source_connection_id\`, \`causation_id\`, \`correlation_id\`, \`payload_hash\`, \`payload\`. If replayed event ID has changed payload: CONFLICT; if same business occurrence repeated from another transport: dedup by canonical occurrence/lifecycle key; do not assert exactly-once delivery.

**Compatibility:** additive optional fields + defaults in minor revision, incompatible semantic/required changes in major revision; consumer versions negotiated; unknown event types quarantined; deprecation window OPEN for Backend agreement. No bus technology approved.

### D. AR-008 — payer / counterparty / outlet (PROPOSED, authority OPEN)

Source semantics: ordinary server generally tracks a client as debtor; contragent server maps clients to salepoints and contragent as payer/debtor. Payments may reference contragent even without an outlet; \`setPayment\` source identifier resolution is context-dependent.

Canonical mapping candidates: **A** CRM/shared Party writer with typed roles and Finance-owned AR attribution; **B** CRM relationship identities plus Finance-owned legal debtor identities with a shared mapping contract. **Recommendation A (PROPOSED)** contingent on CRM, Finance and Sales sign-off. Preserve \`source_server_mode\`, \`external_client\`, \`external_contragent\`, \`outlet_id?\`, \`payer_id\`, \`receivable_owner_id\`, \`party_mapping_evidence\`, \`effective_at\` separately. For non-linked contragent, payer can exist with no outlet. Do not choose payer solely from text \`client\` or from branch.

### E. AR-010 — imported documents and charges (PROPOSED, chargeable trigger OPEN)

**Import processing:** DISCOVERED → STAGED → VALIDATED → MAPPED → ACCEPTED → (owner may POST) → RECONCILED → ARCHIVED; REJECTED, QUARANTINED and CONFLICT side states. \`ACCEPTED\` means imported under owner governance, **not** automatically posted or billable.

**Three independent IDs:** source record identity; canonical ERP document identity; stable business occurrence identity. Fingerprint changes represent revisions, not new economic documents by default. Avoid source import and native creation charging twice. Transport event IDs never serve as sole economic dedup keys.

**Billing event proposal:** Integration does **not** directly debit wallet. Domain owner emits only approved lifecycle fact, via outbox; Billing independently decides chargeable doc types and trigger, with dedup at \`billing_account/tenant + document_id + approved_event_kind + occurrence_id + tariff_version\` (precise final key OPEN). Historic backfill, corrections, re-import, status flip, return/reversal, late sync and closed-period adjustments remain OPEN for Billing+Sales. No automatic charge from STAGED, ACCEPTED, POSTED or external GET alone until lifecycle trigger APPROVED.

### F. Migration and cutover/rollback evidence (PROPOSED)

| Step | Control | Required evidence |
|---|---|---|
| 0 Source inventory | authorized source accounts, provider/filial and server mode | source inventory; scopes; data ownership |
| 1 Read-only extraction | bounded windows + pagination and checksums | endpoint/method/version/time window; counts; raw immutable references |
| 2 Master data | map UoM, products, branches, parties/payers | mapping collisions + operator resolutions |
| 3 Transactions | order/receipt/dispatch/payment/status staging; no direct GL/stock ledger mutation | revision hashes; domain rejections |
| 4 Reconciliation | accounting AR by payer/currency, inventory by warehouse/cutoff, document lifecycle | exceptions, signed reconciliation and as-of zone |
| 5 Parallel observation | source remains system of record; overlap pulls and changes | drift trend, frozen high-watermark, duplicate counts |
| 6 Cutover | explicit single writer, final delta and owner approvals | source writer freeze acknowledgement, domain signoffs, snapshots and rollback plan |
| 7 Post-cutover | preserve audit, disable obsolete credentials only after approval | unresolved exceptions log, rollback window expiry and retention |

Rollback tiers: **pre-posting** discard staging (retain audit); **pre-cutover after posting** use owner-approved compensating transactions (never delete ledger); **post-cutover** forward-repair/controlled re-cutover with economic occurrence dedup and explicit approval. Auto-fallback to both writers is forbidden. No claims of zero downtime, guaranteed full history or tested DR are made.

### G. Failure / security / evidence gates

- Provider request classification: 401 serialized reauth and safe retry; 403 stop (permission), 400/402 validate mapping/method, 407 reduce batch, 429 token bucket and backoff; timeouts/partial SET **ambiguous** → reconcile before replay. Never blanket auto-retry payment, stock or balance SET.
- Track capability evidence record: \`provider\`, \`method\`, \`documentation_url\`, \`doc_revision/checked_at\`, \`evidence_level\`, \`verification_environment\`, \`verification_timestamp\`, \`test_case/response_reference\`, \`reviewer\`, \`limitations\`. All current SalesDoc features = DOCUMENTED, not LIVE_VERIFIED.
- Approved environment/permission is required for sandbox/production validation; no real credentials or writes at architecture stage.
- External request resource usage may be metered only if Resource Usage category is approved. Reconciliation queries do not inherently create billable ERP business documents.
- Source-provider secrets in vault references, TLS, egress allowlist, tenant+branch validation, rate limit budgets, redacted logs and audit evidence. Revoked tenant permissions block subsequent imports/exports.

### H. ADR registry / open questions

Owned ADRs **ADR-INT-001..005 remain PROPOSED**; proposals here refine their intended contract, not an approval. Cross-module OPEN: AR-002 Catalog scope; AR-008 Party writer and Finance debtor; AR-010 billable document trigger and revisions; AR-011 evidence level needed to ship; MR-10 version/deprecation; MR-19 cutover criteria; timezone and archival retention. Only owning module may finalize its rules.

**Requested shared registry synchronization (NOT APPLIED):** DECISIONS register refined ADR-INT-001..005 PROPOSED; REQUIREMENTS add evidence and migration acceptance; MODULE_REGISTRY add canonical import ownership; OPEN_QUESTIONS retain AR-008/010/011; MASTER link provider-neutral ACL and ownership; CHANGELOG record 2026-10-12 Integration v0.3.

## Codex — Integration Specification (ARCHITECTURE ONLY; NO CODE)

**Target:** eventual implementation contract, pending ADR approvals. Design separate logical responsibilities: SalesDocAdapterV2, CredentialBroker, CapabilityEvidenceRegistry, MethodPolicyRateLimiter, SyncOrchestrator, SourceStaging, ExternalIdentityRegistry, CanonicalMapper, DomainImportGateway, ReconciliationEngine, CutoverCoordinator. Initial runtime modular monolith compatible; no premature services.

**Artifacts required for later implementation:** versioned API/event schemas; per-method documented feature registry; normalized operation error taxonomy; permission and tenant isolation matrix; import state transition and idempotency specification; origin/fingerprint/occurrence ledger; branch+party mapping conflict process; reconciliation report contract; rollback runbooks and test acceptance matrix. No code is authorized by this document.

**Must-pass conceptual scenarios:** ordinary and contragent source servers; payment without outlet; same \`code_1C\` in two source branches; CS_id/SD_id collision; 401 and concurrent logins; 403/407/429; partial SET; timeout after possible finance write; duplicated and changed payload; offset page mutation; late offline events; previously posted imported doc re-pull; canceled/returned order; cross-tenant branch spoof; incomplete history and missing deletion feed; signed cutover and compensating rollback. Any scenario without documented source contract remains UNVERIFIED until authorized testing.

---

## Round 2 correction — R2-DOC-02 / MR-19 (2026-10-12)

**PROPOSED; runtime validation: NOT_VERIFIED.** Detailed documented SalesDoc V2 source claims lost between historical [016573c8 baseline](https://github.com/yakhyodev/erp-saas-architecture/blob/016573c8426ed3ea3b1eacbd0dc81adb67861220/docs/modules/INTEGRATION_ARCHITECTURE.md) and later reconciled contracts are restored in the dedicated [SalesDoc V2 Provider Compatibility & Evidence Appendix](SALESDOC_V2_PROVIDER_COMPATIBILITY.md). That appendix records a method/capability matrix, source links, CS_id/SD_id/code_1C precedence, pagination, application error and partial-SET semantics, documented rate limits, ordinary-versus-contragent payer identity, migration/cutover evidence and an API/event/outbox/retry/single-writer safety matrix. Each provider claim is **DOCUMENTED**, not LIVE_VERIFIED; inferred or unknown capabilities are marked INFERRED/UNVERIFIED. Source production and sandbox checks are **NOT_VERIFIED**.

### Architecture preservation and boundary
- Integration remains a provider adapter/staging/identity mapping context, not authoritative writer for Party/Product/SKU, Sales lifecycle, Inventory movement, Finance GL/AR, or Billing rating/wallet.
- CRM/shared Party owner and Product Catalog owner are **OPEN**, not assigned here. Source contragent, payer, outlet and receivable owner must retain separate IDs and provenance; Finance debtor attribution and CRM outlet relationship require their own owner contracts.
- Imported historical source documents are observations until owning domain explicitly accepts them. A source ID plus revision hash distinguishes new revisions from new logical documents. Integration never emits a direct SaaS wallet debit; Billing decides accepted chargeable occurrence, lifecycle trigger, historical charge eligibility and reversal with Sales/Inventory (**AR-010 OPEN**).
- Transport `event_id` is not economic `business_occurrence_id`; owner-assigned business facts use scoped uniqueness, domain transaction + outbox and deduplicating inbox (PROPOSED). Duplicate envelope delivery does not create duplicate posting; same key/different payload is quarantined.
- External write/SET and provider webhooks are not enabled by documentary evidence alone. Signed verification, nonce/timestamp/replay defense and tenant-bound endpoint authorization are PROPOSED controls if a webhook capability is later established. Native SalesDoc webhook support is **UNVERIFIED**.
- Cutover requires source inventory, snapshot/checkpoint, reversible mapping, read-only run, owner reconciliation for stock/debt/documents, explicit single-writer switch and signed rollback/compensation plan. Actual rehearsal and production capability remain **NOT_VERIFIED**.
- Prior paragraphs remain historical PROPOSED hypotheses, not new APPROVED decisions. Canonical details remain subject to Backend/Sales/CRM/Finance/Billing/Security agreement.

**Shared-doc follow-up only (NOT EDITED):** Master/Decisions/Module Registry/Open Questions/Changelog owners should link this provider appendix and retain AR-008, AR-010, AR-011, MR-10 and MR-19 as OPEN until approvals and evidence. No code authorized.
