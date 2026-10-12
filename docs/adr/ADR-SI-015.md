# ADR-SI-015 — Round 2 Typed Lifecycle, Outbox and Billing Occurrence

**Status: PROPOSED** | 2026-10-12 | AR-005, AR-010, MR-07, MR-08

## Muammo
Generic SalesDocument postingning physical StockMovement, ERP Finance GL va SaaS Billing charge sifatida noto‘g‘ri takroran talqin qilinishi double-posting va double-billingga olib keladi.

## Variantlar
A. Sales document, Inventory va Finance GL bitta cross-domain ACID transactionda (Database eski §6 tarixiy alternativa PROPOSED).
B. Har authoritative owner local atomic transaction + transactional outbox, independent inbox/dedup (Database §18 va Backend §18 bilan mos PROPOSED tavsiya).
C. Best-effort API chain no durable outbox (muvaffaqiyatsizlik xavfi yuqori).

## Tanlangan taklif
B: Sales CONFIRM commercial state only, StockMovement yo‘q; Inventory RESERVE physical effect emas; DISPATCH Inventory unique posting leg bo‘yicha bir marta debit+reservation consume; Finance INVOICE jurnali alohida va stockni takror kamaytirmaydi; GoodsReceipt bir marta credit; return/reversal faqat original postingga bog‘langan compensation. Finance COGS/GL recognition and valuation OPEN. Billing faqat non-monetary source lifecycle candidate consumes, chargeable type/transition/refund/partial/import mapping OPEN.

## Idempotency/event
Versioned command: tenant, actor, command_id, expected_revision, idempotency_key, payload fingerprint, correlation. Versioned event: immutable envelope event_id va undan mustaqil stable business_occurrence_id; aggregate revision, original_occurrence_id, occurred_at, server_committed_at. Inbox transport event unique va effect business key unique. Same key/different payload error. Out-of-order recheck; delayed consumer retries do not duplicate physical movement. Inventory local movement+balance+reservation+outbox atomar. Finance va Billing o‘z journal/charge ledgerini alohida atomar qiladi.

## Sabab
Writer isolation, replay safety, future scaling/enterprise isolation va mustaqil accounting policy. Finance/Billing vaqtinchalik kechikishi UI pending status hamda reconciliation orqali ifodalanadi.

## Xavflar va mitigation
At-least-once redelivery, stalled inbox/outbox, stock-vs-GL eventual consistency, import duplicate, invoice double effects; business-effect uniqueness, DLQ review, reversal linkage, discrepancy checks, operator notification. No unmeasured reliability claim.

## Boshqa modullarga ta'siri
Database §18/Backend §18, Finance AR/GL, Billing §20.4, CRM Party refs, Integrations source identity, Mobile offline, UX-T01…T20. [Detailed matrix](../modules/SALES_INVENTORY.md).

## OPEN approval gates
Negative stock exceptions; reservation TTL/renewal; FIFO vs weighted average/lot valuation; stock count cutoff freeze vs as-of; partial delivery and invoice; chargeable document type/trigger, cancel/return/import refund; legal payer mapping and offline month. Domain owners and user must approve; no new business trigger or pricing rule approved.
