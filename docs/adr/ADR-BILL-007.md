# ADR-BILL-007 — Marginal progressive discount tiers

- **Status:** PROPOSED
- **Date:** 2026-10-11
- **Scope:** Billing Architecture
- **Business-rule authority:** APPROVED biznes invariantlari [Billing module](../modules/BILLING_ARCHITECTURE.md) da; bu ADR texnik usulni tasdiqlamaydi.

## Muammo
Progressiv chegirma haddan o'tganda qanday qo'llanadi?

## Ko'rilgan variantlar
All-volume retroactive tier; marginal tier; flat discounts.

## Tanlangan yechim (taklif)
PROPOSED: marginal bracket-by-bracket tiers with saved per-tier breakdown. Progressive discounts themselves APPROVED, mechanism not.

## Sabab
Limitda keskin chegirma sakrashini kamaytiradi, hisob tushunarli.

## Xavflar va ochiq masalalar
Thresholds, discount order, enterprise combination va period OPEN.

## Boshqa modullarga ta'siri
Pricing, Enterprise Contracts, Billing Reports.

## Qaror tartibi
Foydalanuvchi alohida tasdiqlamaguncha PROPOSED. O'zgartirilsa eski ADR SUPERSEDED sifatida tarixda qoladi.

## MR-03 / MR-05 clarification (PROPOSED)
See [Billing §19.5–19.6](../modules/BILLING_ARCHITECTURE.md#196-mr-04--mr-05--volume-resource-discount-and-pricing-conflict-gates). Progressive discount principle APPROVED, marginal tier algorithm PROPOSED only. OPEN: tier window, marginal vs retroactive tiers, seat discount applicability, enterprise override order, stacking and rounding. Record immutable rate components and tier breakdown with each charge; replays and corrections never silently rerate historical posted records.
