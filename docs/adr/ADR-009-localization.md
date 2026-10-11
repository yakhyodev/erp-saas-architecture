# ADR-009: Localization model

- **Status:** PROPOSED
- **Source:** 01 — Master Architecture chat
- **Scope:** umumiy arxitektura, boshqa modulning ichki qarorini almashtirmaydi

## Muammo
Uchta til va yashirin o‘zbek alifbosi butun mahsulotda bir xil ishlashi kerak.

## Variantlar
1. Har sahifada statik strings
2. Key-based central dictionary va locale fallback

## Tavsiya etilgan tanlov (hali APPROVED emas)
Key-based central dictionary va locale fallback (PROPOSED)

## Sabab
Terminlarning izchilligi va yangi alifbo uchun evolyutsiya.

## Xavflar
Translation drift, search normalization collisions.

## Boshqa modullarga ta'siri
Web, Mobile, Search, Reporting

## Cheklov
Tasdiqlangan biznes qoidalar o‘zgartirilmaydi. Tegishli modul va Master Architecture reviewdan so‘ng tasdiqlash kerak. Tafsilotlar uchun [master arxitektura](../modules/MASTER_ARCHITECTURE.md)ga qarang.
