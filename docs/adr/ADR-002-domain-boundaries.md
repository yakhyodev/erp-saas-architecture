# ADR-002: Domain ownership

- **Status:** PROPOSED
- **Source:** 01 — Master Architecture chat
- **Scope:** umumiy arxitektura, boshqa modulning ichki qarorini almashtirmaydi

## Muammo
ERP modullari bir xil hujjat yoki jadval uchun javobgarlikni bo‘lishib yuborishi mumkin.

## Variantlar
1. Shared database logic
2. Domain-owned write model + public contracts

## Tavsiya etilgan tanlov (hali APPROVED emas)
Domain-owned write model + public contracts

## Sabab
Kelajakgi extraction va transactional invariantlar aniq bo‘ladi.

## Xavflar
Cross-domain workflowda qo‘shimcha orchestration talab etiladi.

## Boshqa modullarga ta'siri
Sales, Inventory, Finance, CRM, Reporting

## Cheklov
Tasdiqlangan biznes qoidalar o‘zgartirilmaydi. Tegishli modul va Master Architecture reviewdan so‘ng tasdiqlash kerak. Tafsilotlar uchun [master arxitektura](../modules/MASTER_ARCHITECTURE.md)ga qarang.
