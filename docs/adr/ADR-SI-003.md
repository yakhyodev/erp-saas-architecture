# ADR-SI-003: Parallel qoldiq operatsiyalari

- **Status:** PROPOSED (tasdiqlanmagan)
- **Date:** 2026-10-11
- **Domain:** Sales & Inventory
- **Source:** [SALES_INVENTORY.md](../modules/SALES_INVENTORY.md)

## Muammo
Bir vaqtning o‘zida sotish/rezerv qilish oversellga olib kelishi mumkin.

## Variantlar
Ilova darajasida oldin tekshirish; DB row lock; atomar conditional update.

## Taklif etilgan qaror
DB tranzaksiyasida row lock yoki conditional update, deterministik lock order, idempotency.

## Sababi
TOCTOU va double-postingdan himoya.

## Xavflar va mitigation
Hot SKU contention va deadlock; benchmark, qisqa retry zarur.

## Boshqa modullarga ta’siri
Database, Backend, Mobile Offline.

## Tasdiqlash
Bosh arxitektura, tegishli domen egalari bilan reviewdan keyin APPROVED qilinadi. Hozircha implementation mandate emas.
