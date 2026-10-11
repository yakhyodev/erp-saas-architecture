# ADR-007: Billing domain boundary

- **Status:** PROPOSED
- **Source:** 01 — Master Architecture chat
- **Scope:** umumiy arxitektura, boshqa modulning ichki qarorini almashtirmaydi

## Muammo
Seat, hujjat, resurs usage bir markazda hisoblanmasa tarif va audit nomuvofiq bo‘ladi.

## Variantlar
1. Har bir modul billing hisoblaydi
2. Alohida platform Billing domeni

## Tavsiya etilgan tanlov (hali APPROVED emas)
Alohida platform Billing domeni (PROPOSED)

## Sabab
Usage producerlar fakt beradi, pricing/balance Billingda; SaaS va ERP Finance ledgerlar alohida.

## Xavflar
Event duplicates, delayed usage va negative balance settlement.

## Boshqa modullarga ta'siri
Billing, Tenant, Finance, API, barcha usage producerlar

## Cheklov
Tasdiqlangan biznes qoidalar o‘zgartirilmaydi. Tegishli modul va Master Architecture reviewdan so‘ng tasdiqlash kerak. Tafsilotlar uchun [master arxitektura](../modules/MASTER_ARCHITECTURE.md)ga qarang.
