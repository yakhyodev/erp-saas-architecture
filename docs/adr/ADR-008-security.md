# ADR-008: Security architecture

- **Status:** PROPOSED
- **Source:** 01 — Master Architecture chat
- **Scope:** umumiy arxitektura, boshqa modulning ichki qarorini almashtirmaydi

## Muammo
Cross-tenant data leak va noto‘g‘ri permissions ERPda kritik xavf.

## Variantlar
1. Faqat UI-level permission
2. Central identity + domain enforcement

## Tavsiya etilgan tanlov (hali APPROVED emas)
Central identity + domain enforcement (PROPOSED)

## Sabab
Defense in depth va minimum privileges.

## Xavflar
Policy drift va murakkab company/branch role mapping.

## Boshqa modullarga ta'siri
Security, Tenant, API, UI, Audit

## Cheklov
Tasdiqlangan biznes qoidalar o‘zgartirilmaydi. Tegishli modul va Master Architecture reviewdan so‘ng tasdiqlash kerak. Tafsilotlar uchun [master arxitektura](../modules/MASTER_ARCHITECTURE.md)ga qarang.
