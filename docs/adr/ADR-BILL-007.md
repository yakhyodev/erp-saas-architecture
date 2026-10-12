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

## Round 2 clarification — MR-05 (2026-10-12, PROPOSED)
Compare marginal tiers vs retroactive cumulative tiers, contract net-rate overrides vs enterprise-before-standard-discount stacking, per-company vs account-wide usage and effective-dated price selection. [Billing §20.6–20.7](../modules/BILLING_ARCHITECTURE.md). Recommended review option: marginal tiers and immutable versioned contract precedence; still OPEN. Pricing snapshot includes amount components, unit, threshold, cumulative window, applied version/override, effective time, fixed-point rounding and billed/posted periods. Resource Usage may be event-metered or period-snapshot; categories/rates/included quantities OPEN. No automatic re-rate of posted ledger.

## User-selected tier calculation candidate (2026-10-12, PROPOSED)
Marginal progressive tiers (independent discount per consumption bracket) are **SELECTED_PENDING_APPROVAL**, not APPROVED; progressive discounts as a principle remain APPROVED. OPEN: tier thresholds, cumulative scope/window, marginal rates, retroactive adjustments, enterprise contract override precedence/stacking, seat applicability and fixed-point rounding. Immutable per-tier breakdown, source quantity and price version required once policy confirmed. See [Billing §21](../modules/BILLING_ARCHITECTURE.md).
