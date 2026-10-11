# ADR-001: Tizim chegarasi

- **Status:** PROPOSED
- **Source:** 01 — Master Architecture chat
- **Scope:** umumiy arxitektura, boshqa modulning ichki qarorini almashtirmaydi

## Muammo
Dastlabki servis chegaralari va tashqi integratsiyalar farqlanmasa ownership chalkashadi.

## Variantlar
1. Yagona yopiq monolit
2. Dastlab microservicelar
3. Modular monolith va explicit external boundary

## Tavsiya etilgan tanlov (hali APPROVED emas)
Modular monolith va explicit external boundary

## Sabab
Bir deployable va aniq external adapterlar boshlanish xarajatini kamaytiradi.

## Xavflar
External integration talablarini kech aniqlash.

## Boshqa modullarga ta'siri
Integrations, API, Security, Infra

## Cheklov
Tasdiqlangan biznes qoidalar o‘zgartirilmaydi. Tegishli modul va Master Architecture reviewdan so‘ng tasdiqlash kerak. Tafsilotlar uchun [master arxitektura](../modules/MASTER_ARCHITECTURE.md)ga qarang.
