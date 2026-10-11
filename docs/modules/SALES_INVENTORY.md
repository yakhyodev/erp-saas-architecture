# SALES_INVENTORY.md
ERP SaaS Sales & Inventory Architecture v0.1.0 | 2026-10-11 | **PROPOSED**

## Tasdiqlangan platforma biznes talablari (APPROVED)
Hybrid Prepaid SaaS: Seat + Business Usage (hujjat asosiy narxi, hajmdan keyingi kredit) + Resource Usage; 1 kredit = 10 so‘m; progressiv usage chegirmasi, enterprise shartnoma; balans tugaganda haqiqiy Seat/Usage bir kalendar oy minusga yoziladi va taxminiy kredit limiti yo‘q. Barcha kompaniya va filiallar bitta SaaSda. uz/en/ru hamda yashirin kelajakdagi o‘zbek alifbosi.

## Hozircha tasdiqlanmagan texnik qarorlar (PROPOSED)
Go, PostgreSQL, Next.js, Flutter; modular monolith, RLS/sharding; quyidagi domen obyektlari, statuslari, invariantlari va workflow. SalesDoc API faqat mavjud endpointlar dalili, barcha UI funksiyalari dalili emas.

## Modullar va chegaralar
Sales — buyurtma/order lines va fulfillment; Purchase — PO, kirim/qabul; Warehouse/Inventory — qoldiq, ledger, rezerv, transfer, qaytarish, inventarizatsiya; Catalog — kategoriya, product, SKU, variant, UoM; Pricing — narxlar va snapshot; Document Workflow — approval/posting/reversal. CRM customer, agent, route egasi; Finance ledger/AR/AP/invoice/valuation/refund egasi; Billing SaaS wallet/usage egasi; Security auth va RBAC egasi.

## Tenant va Warehouse
Tenant → Company/Legal Entity → Branch → Warehouse → optional Zone/Bin. Stock yozuvlari tenant scoped, branch/warehouse ruxsati majburiy. Cross-tenant transfer taqiqlanadi; cross-company huquqiy oqimi OPEN. Stock bucketlar SELLABLE, QUARANTINE, DAMAGED, IN_TRANSIT. Lot/serial/owner optional.

## 1. Sales order
SalesOrder: tenant/company/branch/customer, currency, source, order date, revision/status, totals. SalesOrderLine: SKU, ordered/base qty, UoM, price/discount/tax snapshot, reserved, fulfilled, cancelled qty. Proposed DRAFT → SUBMITTED → CONFIRMED → PARTIALLY_FULFILLED → FULFILLED → CLOSED; alternate REJECTED/ON_HOLD/CANCELLED. Order tasdiqlash physical stockni kamaytirmaydi; dispatch kamaytiradi. Qisman fulfillment/backorder. Fulfilled + cancelled ordered qtydan oshmasin (explicit tolerant exception mustasno). Reserve confirmation yoki allocation bosqichida: OPEN.

## 2. Purchase
PurchaseOrder va lines: supplier, SKU, quantities, price, currency, approval/status. DRAFT → APPROVAL_PENDING → APPROVED → PARTIALLY_RECEIVED → RECEIVED → CLOSED. GoodsReceipt: DRAFT → INSPECTED → POSTED. Qabul qilingan tovar tegishli bucketga kiradi, rejected tovar SELLABLEga emas. Partial receipt, supplier return; over-receipt tolerance va 3-way matching OPEN.

## 3. Warehouse
Warehouse ACTIVE/BLOCKED/CLOSED; receiving, pickable, quarantine zonalari; optional bins. Large distributor: multi-warehouse visibility, picking, batch/lot, expiry/FEFO, crossdock va consignment extensibility. Bularning hammasi MVP deb tasdiqlanmagan.

## 4. Stock movements, balances, reservation
Immutable StockMovement: tenant, posting operation ID, source document/line, SKU, warehouse/bin, lot/serial, bucket, signed base qty, posted/effective dates, actor, reason, reversal reference. Hard delete/edit taqiqlanadi; reversal compensation bilan.
StockBalance ayni atomar DB transactionda yangilanadigan tezkor read model. Available = on_hand_sellable - active_reserved. Quarantine/in_transit SELLABLEga kirmaydi. Ledger bilan davriy reconciliation.
StockReservation ACTIVE → CONSUMED/RELEASED/EXPIRED; physical stock emas. Confirm/cancel/dispatch rezervni tegishlicha ajratish, release/consume qiladi.
Posting: authorize → valid state/qty → deterministic balance lock/conditional update → movement+balance+reservation+document+outbox bitta ACID transaction → commit. Same idempotency key va boshqa payload conflict, duplicate posting no stock duplication.

## 5. Transfer
DRAFT → APPROVED → DISPATCHED → PARTIALLY_RECEIVED → RECEIVED → CLOSED. Dispatch source sellable debit, in_transit credit; destination receiving in_transit debit, accepted bucket credit. Dispatched = received + remaining_in_transit + documented_loss. Partial receive va discrepancy audit; dispatchdan keyin oddiy cancel emas, compensating workflow.

## 6. Returns
Customer return original dispatch line bilan reference: authorize, receive, inspect, post to quarantine/sellable. Refund/credit note Finance egasi. Supplier return original receipt bilan reference, stock decrement va Finance debit note. Original qtydan ko‘p qaytarilmasin, concurrency-safe. No-reference return OPEN.

## 7. Inventory count
CountSession: warehouse/scope/cutoff/snapshot, count lines, recount/approval. PLANNED → COUNTING → REVIEW → APPROVED → POSTED. Adjustment = counted at cutoff - theoretical at cutoff, movement sifatida audit. Parallel movementsni snapshot reconciliation yoki bin freeze bilan hal qilish usuli OPEN.

## 8. Price list
Versioned PriceList(scope, currency, effective dates, priority) va PriceListItem(SKU, unit, min qty tier, price, constraints). Precedence taklifi: customer negotiated > customer segment > branch/channel > tenant default; equal priority resolution deterministic. Confirmationda price, UoM conversion, discount/tax snapshot immutable. SaaS Business Usage discounts boshqa domen.

## 9. Product hierarchy
Category tree → Product family → Product → SKU/Variant. SKU immutable ID; barcode, supplier mapping, active flag. Base UoM qoldiq birligi; exact decimal/rational conversion (dona/quti, kg, litr), snapshot historical docsda. Floating-point qoldiq/money ishlatilmasin. Lot/expiry/serial va FEFO OPEN.

## 10. Document workflow & invariants
Hujjatlarda tenant, type, ID, number, revision, actor, dates, approval/status, idempotency/correlation. Approve va Post turli commandlar; posted document silent-edit/hard-delete qilinmaydi. Physical stock faqat explicit dispatch/receive/postda o‘zgaradi. Default available negative emas; exception policy OPEN, SaaS wallet minus-balansi ombor minusiga ruxsat emas. RBAC branch/warehouse bilan, UTC timestamps va tenant business date, offline stock server authoritative. Outbox events at-least-once va subscriber dedup.

## Events va integratsiyalar
SalesOrderConfirmed, StockReserved, StockDispatched, GoodsReceived, TransferDispatched, TransferReceived, ReturnPosted, StockAdjusted, PriceListActivated. Billing chargeable document occurrence va reversal eventni alohida tasdiqlaydi. ERP Finance valuation/invoicing alohida. CRM agent/customer linking. Error codes: INSUFFICIENT_STOCK, INVALID_STATE, DUPLICATE_OPERATION, TENANT_SCOPE_DENIED, COUNT_STALE.

## ADRlar — barchasi PROPOSED
- [ADR-SI-001](../adr/ADR-SI-001.md) Domen chegaralari
- [ADR-SI-002](../adr/ADR-SI-002.md) Immutable stock ledger
- [ADR-SI-003](../adr/ADR-SI-003.md) Parallel stock operations
- [ADR-SI-004](../adr/ADR-SI-004.md) Reservation
- [ADR-SI-005](../adr/ADR-SI-005.md) In-transit transfer
- [ADR-SI-006](../adr/ADR-SI-006.md) Price snapshot
- [ADR-SI-007](../adr/ADR-SI-007.md) Posted document immutability
- [ADR-SI-008](../adr/ADR-SI-008.md) Stocktaking cutoff
- [ADR-SI-009](../adr/ADR-SI-009.md) Tenant scope
- [ADR-SI-010](../adr/ADR-SI-010.md) Base UoM conversion

## OPEN QUESTIONS
Negative stock policy; reserve TTL and trigger; FIFO/weighted average valuation; multi-warehouse/intercompany; lot/serial/FEFO scope; no-reference return; inventory count cutoff strategy; currency/discount stacking; offline POS conflict; landed cost; billing document recognition/reversal; shared warehouse rights; full SalesDoc API coverage; SLO targets.

## Umumiy hujjatlar uchun alohida taklif (o‘zgartirilmagan)
DECISIONS.md: 10 PROPOSED ADR; MASTER_ARCHITECTURE.md: ownership and stock invariants; REQUIREMENTS.md: non-oversell, audit, idempotency and partial flows; MODULE_REGISTRY.md: boundaries/events; OPEN_QUESTIONS.md: unresolved policies; CHANGELOG.md: 2026-10-11 v0.1 proposal. Ushbu umumiy fayllarga avtomatik yozilmadi.

## Codex Specification — NO CODE
Approved ADRlardan oldin implementation boshlanmasin. Domain objects: Product/SKU/UoM, PriceList/Item, SalesOrder/Line, PurchaseOrder/Line, GoodsReceipt/Line, Warehouse/Location, StockBalance/Movement/Reservation, StockTransfer/Line, Customer/SupplierReturn, CountSession/Line, DocumentTransition, AuditEvent, OutboxEvent.
Commands: draft/create/edit, submit/confirm/approve, reserve/release/consume, dispatch, receive/post, inspect/return, count/recount/adjust, resolve price, reverse.
Har commandda tenant, actor, expected revision, idempotency, authorization, validation, error va domain events.
Acceptance tests: concurrent reserve 80 + 40 on 100 only one succeeds; duplicate posting exactly one physical effect; transfer 50 receive 35 -> transit 15; order 10 fulfill 6 cancel 4; customer return quarantine; price snapshot unaffected by later price list change; cross-tenant forbidden; parallel stock/count consistency; offline replay idempotent; reversal preserves audit and triggers agreed Billing event. After approval: API contract, conceptual schema, state diagrams, permission/event and concurrency tests. Kod yozilmasin.


## 11. Cross-module harmonization: AR-002, AR-005, AR-010; MR-07/MR-08
**Status: PROPOSED.** Asos: main dagi DECISIONS.md, REQUIREMENTS.md, MODULE_REGISTRY.md, OPEN_QUESTIONS.md va tarixiy docs/reviews/ARCHITECTURE_REVIEW.md. Tarixiy audit CURRENT APPROVAL emas. CRM ADR-Sales product ownership va Backend/Catalog bounded context takliflari AR-002 nomuvofiqligi bo‘lib qoladi.

### 11.1 Canonical ownership matrix (AR-002, MR-07)
| Concept | Proposed single write authority | Consumer/contract | OPEN boundary |
|---|---|---|---|
| Category, Product, SKU/variant, barcode, UoM/conversion | Catalog bounded context (05 Sales & Inventory ichida mantiqan alohida yoki mustaqil modul) | Sales/Purchase/Inventory/Pricing/CRM read public SKU contract, never private table writes | Global vs tenant/company catalog, namespace, SKU mapping |
| Price List, price rule and price quote | Pricing context | Sales receives versioned resolved price snapshot; CRM supplies customer-specific applicability | Dynamic contract/discount precedence |
| Sales Order/Line, commercial state, order fulfillment intent | Sales context | CRM/Finance/Billing/Inventory consume references/events | Multi-warehouse orchestration |
| Physical StockMovement, StockBalance, reservation, transfer, stock count | Inventory context | Sales/Purchasing command gateway and events | Quantity versus cost/valuation responsibility |
| Purchase Order/Line, receiving intent, supplier return instruction | Purchasing context | Inventory posts physical receipts/returns; Finance invoices/AP | Receipt without PO, tolerance |
| Customer assortment, outlet assignment, agent/route, customer segment | CRM & Distribution context | Catalog SKU reference, Pricing segment input, Sales customer ref | Canonical Party/Counterparty owner AR-008 |
| ERP GL, AR/AP, financial invoice, cost/valuation journals | Finance | Dispatch/receipt economic facts consumed idempotently | Cost method, journal timing |
| SaaS rating, charge ledger, wallet/usage and refunds | Billing | Non-monetary document lifecycle facts from source owner | Exact billable transition AR-010; payer scope AR-001 |
| Authentication, tenant membership and scoped authorization | Security/Multi-Tenant (joint decision OPEN) | Sales/Inventory enforces checks | AR-013 |

**Alternatives (OPEN):** A tenant-scoped catalog (simpler isolation, duplicate SKU master); B shared global catalogue with tenant product mapping (reuse, complex merge/access); C shared core + company overlays. Organizational deployment (Catalog inside 05 vs separate service) is also OPEN; modular monolith itself remains PROPOSED. Product ID != SKU ID; document line retains SKU and immutable UoM/price snapshot. CRM cannot become a second SKU mutator by merely referencing products. No existing CRM ADR is modified here.

### 11.2 Transition-to-effects contract (AR-005, MR-08)
**Columns are distinct facts.** N = no effect proposed; P = proposed effect; OPEN = unapproved Accounting/Billing business trigger. Billing candidate fact is **NOT** a charge. Invoice is Finance-owned accounting document, unless an eventual domain decision says otherwise.

| Action / transition | Document/state owner | Inventory physical stock / reservation | Finance GL fact | Billing usage |
|---|---|---|---|---|
| Sales DRAFT CREATE/EDIT | Sales | N / N | N | OPEN candidate; no automatic charge |
| Sales CONFIRM | Sales | N physical / optional RESERVE only if policy approved | N under basic model, commitment accounting OPEN | OPEN confirm candidate |
| RESERVE/RELEASE/EXPIRE | Inventory | N physical / active_reserved changes P | N | OPEN, default no classification |
| PICK/PACK | Sales workflow, Inventory allocates | N physical / possible allocation changes | N | OPEN picking document candidate |
| DISPATCH POST | Sales dispatch intent, Inventory movement | P: sellable debit + reservation consume, one posting per leg | OPEN valuation/COGS event and posting timing | OPEN dispatch candidate |
| SALES INVOICE POST | Finance | N (must never dispatch stock twice) | OPEN: AR/revenue/tax posting under Finance policy | OPEN invoice candidate |
| PURCHASE PO CREATE/APPROVE | Purchasing | N physical / N reservation | N; procurement commitment OPEN | OPEN PO candidate |
| GOODS RECEIPT POST | Purchasing + Inventory | P: accepted stock +; damaged/quarantine segregated | OPEN: inventory valuation/GRNI Finance event | OPEN receipt candidate |
| PURCHASE INVOICE POST | Finance | N (must not duplicate goods receipt) | OPEN AP/accrual/matching | OPEN invoice candidate |
| CUSTOMER RETURN RECEIVE/POST | Returns + Inventory | P: accepted returned qty + to appropriate bucket | OPEN reversal/credit note/cost | OPEN return/refund mapping |
| SUPPLIER RETURN DISPATCH | Purchasing + Inventory | P: source debit | OPEN AP/debit note/valuation reversal | OPEN return candidate |
| CANCEL before physical posting | Owning document context | N physical; release existing reservations P | N unless prior Finance posting exists | OPEN cancel policy |
| REVERSE after physical posting | Original owner instructs Inventory | P compensating StockMovement **only if original movement exists** | OPEN separate Finance reversal when original GL exists | OPEN separate adjustment; never assume refund |
| TRANSFER DISPATCH/RECEIVE | Inventory | P source-to-transit / transit-to-destination, each once | OPEN intercompany valuation | OPEN transfer vs transfer leg |
| STOCK COUNT ADJUSTMENT POST | Inventory | P signed adjustment | OPEN inventory gain/loss GL event | OPEN count adjustment charge |

**AR-005 consistency proposal:** Confirmation is a commercial state and does not itself decrement physical stock. Reservation affects available quantity, not on-hand. Dispatch is the physical reduction. Invoice issue does not physically dispatch again. This is not a claim of approved Finance recognition time. A generic SalesDocumentPosted event must never be interpreted as permission for duplicate stock or GL effects. Event names should be typed by actual fact (e.g. SalesOrderConfirmed, StockDispatchPosted, GoodsReceiptPosted, FinanceInvoicePosted, ReversalPosted). 

**Atomicity boundary:** proposed Inventory local transaction atomically writes movement + materialized balance + reservation transition + stock source state + transactional outbox; Finance owns its own ledger posting and durable dedup/inbox. Same-DB atomic multi-context GL posting vs event/outbox asynchronous Finance posting remains **OPEN** for Backend/Database/Finance; do not assert a cross-service distributed transaction. Downstream failure requires retries/observability/reconciliation, not repeat stock debit.

### 11.3 Authority and idempotency keys
| Fact | Authority | Proposed scoped unique identity and dedup |
|---|---|---|
| Document transition | Sales/Purchasing/Returns document owner | tenant_id + document_type + document_id + transition_id (+ revision concurrency guard) |
| Reservation state | Inventory | tenant_id + reservation_id + action_id, immutable state version |
| Physical movement / dispatch / receipt | Inventory | tenant_id + posting_operation_id + movement_leg_id / line_id; exact-once physical effect under retries |
| Accounting journal | Finance | tenant_id + source_fact_id + journal_kind + accounting_policy_version; unique source posting, compensating original ref |
| Non-monetary potential charge occurrence | Source document owner | tenant_id + source_system + document_type + canonical_doc_id + lifecycle_occurrence_id |
| Rated actual charge | Billing | billing_account_id (AR-001 OPEN) + accepted_occurrence_id + tariff_version + charge_kind |
| Reversal/correction | Domain owning original, Finance/Billing owning their separate reversals | original_fact_id + reversal_id + effect_leg; cannot over-reverse |

Client idempotency key and canonical fact ID are different. A client key scoped by tenant/source/command maps to payload hash and stored outcome: same payload retry gives same result; different payload is conflict. Event envelope includes tenant_id, event_id, source_fact_id, aggregate_id/version, correlation/causation, occurred_at, accepted_at, published_at. Consumers tolerate duplicated and out-of-order delivery. Offline occurrence vs receipt/rating billing period = AR-009 OPEN.

### 11.4 Chargeable document lifecycle — AR-010 remains OPEN
**Only APPROVED:** per-document Business Usage base price, volume-after-threshold extra credits, 1 credit = 10 UZS, progressive/enterprise discount, actual Seat/Usage costs may run minus for ONE calendar month without estimated credit ceiling. **Not approved:** which documents or actions are billable, price values, limits, refunds, exception windows or attribution month.

| Option | Potential billing trigger | Tradeoff |
|---|---|---|
| A: Create | DRAFT CREATED | early usage recognition; spam/abandoned drafts risk |
| B: Confirm | unique CONFIRM | meaningful approval; cancellation ambiguity |
| C: Post | specified POST per canonical document | economic action; partial legs can double charge |
| D: Finalize | unique CLOSE/FINALIZE | fewer abandoned drafts; late charge |

**PROPOSED preference, subject to Billing/user approval:** source owner emits stable non-monetary candidate occurrence tied to canonical document identity and a configured transition; Billing alone classifies, rates and records actual usage. Decide per type: SalesOrder, Dispatch, PurchaseOrder, GoodsReceipt, SalesInvoice, CustomerReturn, SupplierReturn, Transfer (header vs each leg), StockCountAdjustment, ImportedHistoricalDocument, Amendment. Each tariff definition must explicitly set eligible transitions, quantity/volume metric, base-fee uniqueness, free/excluded type, idempotency, cancel/reversal/credit adjustments and tenant/branch attribution. A split dispatch does **not** imply repeated base document charge unless approved tariff says so. No assumed refund on reversal.

**Scenarios to approve:** cancel pre/post charge; partial fulfillment; amendment of canonical ID; split shipment; document retries; imported historical documents and duplicate external IDs; offline upload in following month; return linked to original; successful stock post with deferred Finance GL; reversal where original was never billed. Billing determines rating and posting of **real** expenses; Sales does not hold an estimated negative credit limit.

### 11.5 Specific unresolved decision packets
- **Reservation TTL:** Option A no TTL/manual release; B configurable fixed TTL with renewal; C priority-aware adaptive TTL. Proposed provisional expiry with audit, but no expiry of consumed reservation; actual trigger, values, extension and offline conditions OPEN (ADR-SI-004).
- **Inventory valuation:** FIFO / weighted average / lot-specific; cost ownership and COGS/revenue recognition OPEN with Finance. Physical movement is not automatically GL fact.
- **Negative stock:** hard prohibit (current default proposal) vs permissioned exception by tenant/warehouse/SKU; no approval. Billing negative wallet is distinct.
- **Catalog:** tenant SKU vs global canonical + mapping, CRM assortment separation OPEN.
- **Party/payer:** AR-008 and AR-001 remain OPEN; doc references scoped IDs only.
- **Technology:** Go/PostgreSQL/Next.js/Flutter and modular monolith remain PROPOSED.

### 11.6 Test / approval gates for Codex — PROPOSED, NO CODE
Confirm order -> zero physical movement; reserve -> available drops but on-hand unchanged; dispatch once -> one physical debit; invoice -> no second debit; cancel before dispatch -> release; reversal after dispatch -> compensating debit/credit linked to original; purchase receipt -> one physical credit; duplicate GL event -> one Finance journal; duplicate billable fact -> no second actual charge **if tariff defines chargeability**; split dispatch -> base charge count defined only by approved tariff; wrong tenant denied; reservation expiry never changes historical physical movement; offline late occurrence attribution left undecided pending AR-009.
Owner approvals: 05+07+04 for AR-002, 05+06+03+04 for AR-005, 01+05+12 for AR-010, 01+09 for delayed period. **AR-002/005/010 MR-07/08 are clarified but remain OPEN and not closed.**
New proposed ADRs: [ADR-SI-011](../adr/ADR-SI-011.md), [ADR-SI-012](../adr/ADR-SI-012.md), [ADR-SI-013](../adr/ADR-SI-013.md).

### 11.7 Global synchronization suggestions — not applied
DECISIONS: register ADR-SI-011/012/013 as PROPOSED; REQUIREMENTS: add distinct facts/no-double-post acceptance; MODULE_REGISTRY: publish ownership proposals with OPEN label; OPEN_QUESTIONS: retain AR-002/005/010 and record catalog, GL, Billing, TTL/valuation/negative stock choices; CHANGELOG: document harmonization. Historical audit baseline unchanged.
