# ADR-006: Database tanlovi

- **Status:** PROPOSED
- **Source:** 01 — Master Architecture chat
- **Scope:** umumiy arxitektura, boshqa modulning ichki qarorini almashtirmaydi

## Muammo
ERP transaction va ledger invariants uchun transactional storage talab qilinadi.

## Variantlar
1. PostgreSQL
2. Boshqa relational DB
3. Distributed-first data stores

## Tavsiya etilgan tanlov (hali APPROVED emas)
PostgreSQL (PROPOSED)

## Sabab
ACID, mature relational constraints va operatsion ekotizim.

## Xavflar
Hot tenants, locking, partition va sharding murakkabliklari.

## Boshqa modullarga ta'siri
DB, Billing, Inventory, Finance, Infra

## Cheklov
Tasdiqlangan biznes qoidalar o‘zgartirilmaydi. Tegishli modul va Master Architecture reviewdan so‘ng tasdiqlash kerak. Tafsilotlar uchun [master arxitektura](../modules/MASTER_ARCHITECTURE.md)ga qarang.
