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

## 2026-10-11 harmonization (PROPOSED)
AR-002/MR-07: Logical bounded context owners: Catalog mutates Product/SKU/UoM, Pricing mutates prices, Sales owns order, Purchasing owns PO, Inventory owns physical stock, CRM owns assortment/customer distribution relationships. This refines, but does not approve, the modular monolith suggestion: Go/PostgreSQL/Next.js/Flutter/modular monolith still PROPOSED. Catalog scope global vs tenant/company, dedicated module vs inner bounded context OPEN. See [ADR-SI-011](ADR-SI-011.md) and [Sales §11](../modules/SALES_INVENTORY.md). CRM prior ADR remains unchanged; cross-owner approval required.
