# ADR-SI-002 — Immutable Stock Movement Ledger
Status: **PROPOSED** | Date: 2026-10-11 | Module: Sales & Inventory

## Muammo
Tez qoldiq so‘rovlari uchun performance, harakatlarni tekshirish uchun o‘zgarmas audit kerak.

## Variantlar
1. Faqat mutable stock balance.
2. Faqat movement summasi orqali hisoblash.
3. Immutable movement + transactionally maintained balance.

## Taklif qilingan qaror
3-variant. StockMovement append-only truth; StockBalance posting bilan bir DB transactionda yangilanadi. Reversal kompensatsion movement. Reservation physical stock movement emas.

## Sabab
Audit va yuqori hajmli availability query bir vaqtda amalga oshadi.

## Xavflar
Drift: scheduled reconciliation va mismatch alert; hot SKU concurrency: DB locking, idempotency, deadlock retry.

## Ta’sir
Database, Reporting, Finance, Backend, Security, Billing. Tegishli domen egalaridan tasdiq olinmaguncha APPROVED emas.
