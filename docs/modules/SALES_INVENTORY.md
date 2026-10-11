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
