# ADR-005: Multi-tenant strategiyasi

- **Status:** PROPOSED
- **Source:** 01 — Master Architecture chat
- **Scope:** umumiy arxitektura, boshqa modulning ichki qarorini almashtirmaydi

## Muammo
Kompaniyalar uchun yagona SaaS bilan tenant security va enterprise talablari muvozanati.

## Variantlar
1. Shared DB/shared schema
2. Schema per tenant
3. DB per tenant
4. Boshlang‘ich shared infrastructure, keyingi dedicated variant

## Tavsiya etilgan tanlov (hali APPROVED emas)
Boshlang‘ich shared infrastructure, keyingi dedicated variant (aniq isolation OPEN QUESTION)

## Sabab
Ommaviy tenantlar uchun samaradorlik va dedicated growth yo‘li.

## Xavflar
RLS va tenant resolver tanlovi hali ochiq; izolyatsiya xatosi xavfli.

## Boshqa modullarga ta'siri
Tenant, Security, DB, Infra, Billing

## Cheklov
Tasdiqlangan biznes qoidalar o‘zgartirilmaydi. Tegishli modul va Master Architecture reviewdan so‘ng tasdiqlash kerak. Tafsilotlar uchun [master arxitektura](../modules/MASTER_ARCHITECTURE.md)ga qarang.
