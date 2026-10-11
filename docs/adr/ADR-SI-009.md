# ADR-SI-009: Tenant va filial izolatsiyasi

- **Status:** PROPOSED (tasdiqlanmagan)
- **Date:** 2026-10-11
- **Domain:** Sales & Inventory
- **Source:** [SALES_INVENTORY.md](../modules/SALES_INVENTORY.md)

## Muammo
Bir tenantning qoldig‘i boshqasiga sizib chiqmasligi kerak.

## Variantlar
Faqat app filter; DB/RLS defense-in-depth; per-tenant DB.

## Taklif etilgan qaror
Har yozuvda tenant key, authorization va DB-level qo‘shimcha himoya; RLS qarori umumiy Security/Multi-Tenant bilan.

## Sababi
Ko‘p tenant uchun xavfsiz chegaralar.

## Xavflar va mitigation
RLS bilan operational complexity, cross-company logistics.

## Boshqa modullarga ta’siri
Multi-Tenant, Security, Database.

## Tasdiqlash
Bosh arxitektura, tegishli domen egalari bilan reviewdan keyin APPROVED qilinadi. Hozircha implementation mandate emas.
