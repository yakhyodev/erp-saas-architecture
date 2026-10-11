# ADR-004: Evolution va scaling

- **Status:** PROPOSED
- **Source:** 01 — Master Architecture chat
- **Scope:** umumiy arxitektura, boshqa modulning ichki qarorini almashtirmaydi

## Muammo
Growth ko‘payganda arxitektura evolyutsiyasi oldindan o‘ylanishi kerak.

## Variantlar
1. Barchasini darhol microservice qilish
2. Monolithni o‘lchov asosida scale-out va selective extraction

## Tavsiya etilgan tanlov (hali APPROVED emas)
O‘lchov asosida scale-out va selective extraction

## Sabab
Kichik operatsion footprint, keyinchalik mustaqil scale.

## Xavflar
Data consistency, schema migration va tracing narxi.

## Boshqa modullarga ta'siri
Infra, DB, Backend, Audit

## Cheklov
Tasdiqlangan biznes qoidalar o‘zgartirilmaydi. Tegishli modul va Master Architecture reviewdan so‘ng tasdiqlash kerak. Tafsilotlar uchun [master arxitektura](../modules/MASTER_ARCHITECTURE.md)ga qarang.
