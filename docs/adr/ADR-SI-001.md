# ADR-SI-001: Domen chegaralari

- **Status:** PROPOSED (tasdiqlanmagan)
- **Date:** 2026-10-11
- **Domain:** Sales & Inventory
- **Source:** [SALES_INVENTORY.md](../modules/SALES_INVENTORY.md)

## Muammo
Sales, Purchase, Inventory, Pricing va Catalog jarayonlari bir-biriga kuchli bog‘langan.

## Variantlar
Bitta yirik modul; bounded contextli modular monolith; boshidan microservice.

## Taklif etilgan qaror
Modular monolith ichida alohida bounded contextlar; Inventory o‘z ledgerini boshqaradi.

## Sababi
Atomar stock posting saqlanadi va mikroservisga erta o‘tish xarajati yo‘q.

## Xavflar va mitigation
Chegaralar noto‘g‘ri ajratilsa coupling; event kontraktlarini versionlash zarur.

## Boshqa modullarga ta’siri
Backend, Database, CRM, Finance, Billing.

## Tasdiqlash
Bosh arxitektura, tegishli domen egalari bilan reviewdan keyin APPROVED qilinadi. Hozircha implementation mandate emas.
