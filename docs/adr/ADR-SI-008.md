# ADR-SI-008: Inventory count cutoff

- **Status:** PROPOSED (tasdiqlanmagan)
- **Date:** 2026-10-11
- **Domain:** Sales & Inventory
- **Source:** [SALES_INVENTORY.md](../modules/SALES_INVENTORY.md)

## Muammo
Inventarizatsiya vaqtida parallel movementlar tafovutni noto‘g‘ri hisoblatishi mumkin.

## Variantlar
Omborni freeze qilish; bin freeze; as-of snapshot+movement reconciliation.

## Taklif etilgan qaror
Snapshot+movement reconciliation afzal taklif; cutoff mexanizmi tasdiqlanmagan.

## Sababi
Katta omborda butun ishni to‘xtatmaslik mumkin.

## Xavflar va mitigation
Race condition va inventory count integrity; yakuniy algoritm OPEN.

## Boshqa modullarga ta’siri
Database, Warehouse, Security.

## Tasdiqlash
Bosh arxitektura, tegishli domen egalari bilan reviewdan keyin APPROVED qilinadi. Hozircha implementation mandate emas.
