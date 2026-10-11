# ADR-003: Modular monolith

- **Status:** PROPOSED
- **Source:** 01 — Master Architecture chat
- **Scope:** umumiy arxitektura, boshqa modulning ichki qarorini almashtirmaydi

## Muammo
Minglab tenantlar uchun rivojlantirish qulayligi va operatsion soddalik muvozanati kerak.

## Variantlar
1. An'anaviy tightly-coupled monolith
2. Microservices from day one
3. Modular monolith

## Tavsiya etilgan tanlov (hali APPROVED emas)
Modular monolith

## Sabab
Erta distributed complexitydan qochadi.

## Xavflar
Chegaralar buzilsa distributed monolithga o‘xshash muammo qoladi.

## Boshqa modullarga ta'siri
Backend, Deployment, barcha domenlar

## Cheklov
Tasdiqlangan biznes qoidalar o‘zgartirilmaydi. Tegishli modul va Master Architecture reviewdan so‘ng tasdiqlash kerak. Tafsilotlar uchun [master arxitektura](../modules/MASTER_ARCHITECTURE.md)ga qarang.
