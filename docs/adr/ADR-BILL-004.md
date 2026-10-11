# ADR-BILL-004 — Immutable pricing versions

- **Status:** PROPOSED
- **Date:** 2026-10-11
- **Scope:** Billing Architecture
- **Business-rule authority:** APPROVED biznes invariantlari [Billing module](../modules/BILLING_ARCHITECTURE.md) da; bu ADR texnik usulni tasdiqlamaydi.

## Muammo
Tarif o'zgarishi eski charge/invoice'ni qayta narxlab yubormasligi kerak.

## Ko'rilgan variantlar
Mutable price table; immutable effective-dated pricing versions; external rating.

## Tanlangan yechim (taklif)
PROPOSED: immutable time-effective pricing version, deterministic charge breakdown, enterprise override references.

## Sabab
Tarixiy audit, izohlanadigan hisoblash va qayta hisoblash reproducibility.

## Xavflar va ochiq masalalar
Time boundaries, enterprise precedence, retroactivity va rounding OPEN.

## Boshqa modullarga ta'siri
Billing, Enterprise Contracts, Reporting, Finance.

## Qaror tartibi
Foydalanuvchi alohida tasdiqlamaguncha PROPOSED. O'zgartirilsa eski ADR SUPERSEDED sifatida tarixda qoladi.
