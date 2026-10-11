# ADR-SI-010: Base UoM va conversion

- **Status:** PROPOSED (tasdiqlanmagan)
- **Date:** 2026-10-11
- **Domain:** Sales & Inventory
- **Source:** [SALES_INVENTORY.md](../modules/SALES_INVENTORY.md)

## Muammo
Quti/dona/kg/litr va fraksion birliklarda rounding drift xavfi.

## Variantlar
Float; precise decimal/rational; faqat bir unit.

## Taklif etilgan qaror
SKU uchun base UoM, exact conversion, hujjatda conversion snapshot.

## Sababi
Qoldiq va qayta hisoblash aniqligi.

## Xavflar va mitigation
Catch weight, serial/lot, legacy mapping murakkab.

## Boshqa modullarga ta’siri
Catalog, Purchase, Sales, Finance.

## Tasdiqlash
Bosh arxitektura, tegishli domen egalari bilan reviewdan keyin APPROVED qilinadi. Hozircha implementation mandate emas.
