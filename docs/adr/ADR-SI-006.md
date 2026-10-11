# ADR-SI-006: Narx snapshoti

- **Status:** PROPOSED (tasdiqlanmagan)
- **Date:** 2026-10-11
- **Domain:** Sales & Inventory
- **Source:** [SALES_INVENTORY.md](../modules/SALES_INVENTORY.md)

## Muammo
Keyingi narx o‘zgarishi tarixiy order va hisob-kitobni buzmasligi kerak.

## Variantlar
Har doim live price; confirmda snapshot.

## Taklif etilgan qaror
Confirmation paytida versioned price, unit conversion, tax/discount snapshot.

## Sababi
Takror hisobot va audit deterministik.

## Xavflar va mitigation
Valyuta va qayta narxlash policy ochiq.

## Boshqa modullarga ta’siri
Pricing, Finance, CRM, Billing.

## Tasdiqlash
Bosh arxitektura, tegishli domen egalari bilan reviewdan keyin APPROVED qilinadi. Hozircha implementation mandate emas.
