# ADR-SI-007: Posted hujjat immutability

- **Status:** PROPOSED (tasdiqlanmagan)
- **Date:** 2026-10-11
- **Domain:** Sales & Inventory
- **Source:** [SALES_INVENTORY.md](../modules/SALES_INVENTORY.md)

## Muammo
Postingdan keyin tahrir/del ikki marta stock effektiga sabab bo‘lishi mumkin.

## Variantlar
Mutable posted docs; compensation/reversal docs.

## Taklif etilgan qaror
Posted document immutable; korreksiya kompensatsion hujjat bilan.

## Sababi
Audit trail saqlanadi va billing charge mapping aniqlashtiriladi.

## Xavflar va mitigation
Reversal UX va bog‘liq accounting correction murakkab.

## Boshqa modullarga ta’siri
Finance, Billing, Audit, UX.

## Tasdiqlash
Bosh arxitektura, tegishli domen egalari bilan reviewdan keyin APPROVED qilinadi. Hozircha implementation mandate emas.
