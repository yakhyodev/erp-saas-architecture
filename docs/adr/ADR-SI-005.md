# ADR-SI-005: Ikki bosqichli transfer

- **Status:** PROPOSED (tasdiqlanmagan)
- **Date:** 2026-10-11
- **Domain:** Sales & Inventory
- **Source:** [SALES_INVENTORY.md](../modules/SALES_INVENTORY.md)

## Muammo
Yo‘ldagi tovar ikki omborda hisoblanmasligi kerak.

## Variantlar
Bir vaqtda debit-credit; dispatched/in-transit/received.

## Taklif etilgan qaror
Dispatchda source debit+in_transit credit; receive'da transit debit+destination credit.

## Sababi
Qisman qabul, yo‘qolish va kechikish kuzatiladi.

## Xavflar va mitigation
Yopilmagan transitlar; reconciliation jarayoni zarur.

## Boshqa modullarga ta’siri
Inventory, Finance, CRM Distribution.

## Tasdiqlash
Bosh arxitektura, tegishli domen egalari bilan reviewdan keyin APPROVED qilinadi. Hozircha implementation mandate emas.
