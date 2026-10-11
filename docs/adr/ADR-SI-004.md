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

## 2026-10-11 reservation policy clarification (PROPOSED)
A: manual release/no expiry; B: configurable reservation TTL plus renewal; C: priority-aware adaptive expiry. No fixed TTL, priority, allocation trigger or offline extension APPROVED. Reservation acts on available, not physical on-hand; expiry/release must be atomic/idempotent and must never reverse a consumed/dispatched reservation. OPEN: auto reserve at confirm vs pick allocation; TTL, grace, reallocation fairness, disconnected device and warehouse scope. See [Sales §11](../modules/SALES_INVENTORY.md).
