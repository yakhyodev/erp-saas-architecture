# ADR-SI-004: Reservation modeli

- **Status:** PROPOSED (tasdiqlanmagan)
- **Date:** 2026-10-11
- **Domain:** Sales & Inventory
- **Source:** [SALES_INVENTORY.md](../modules/SALES_INVENTORY.md)

## Muammo
Buyurtma tovarni band qilishi kerak, lekin jismoniy kirim-chiqim emas.

## Variantlar
Reserve vaqtida physical decrement; alohida reservation lifecycle.

## Taklif etilgan qaror
ACTIVE/CONSUMED/RELEASED/EXPIRED reservation; available=on_hand_sellable-active_reserved.

## Sababi
Physical stock va sotuvi mumkin qoldiq farqlanadi.

## Xavflar va mitigation
Stale rezervlar, TTL/priority ochiq.

## Boshqa modullarga ta’siri
Sales, Warehouse, Mobile.

## Tasdiqlash
Bosh arxitektura, tegishli domen egalari bilan reviewdan keyin APPROVED qilinadi. Hozircha implementation mandate emas.
