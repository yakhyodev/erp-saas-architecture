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

## MR-04 / MR-05 and late event clarification (PROPOSED)
See [Billing §19.5–19.6](../modules/BILLING_ARCHITECTURE.md#196-mr-04--mr-05--volume-resource-discount-and-pricing-conflict-gates). Pricing snapshot includes eligibility policy, effective-dated contract, base/over-threshold volume rule, resource unit, discount window/tier algorithm, rounding, and original/posted period links. **OPEN** whether late offline event uses server-effective, received, validated device occurrence, or controlled adjustment period; choosing price version cannot precede this policy. Enterprise precedence and stackability are OPEN; no invented numerical tiers or discount.

## User-selected effective-period clarification (2026-10-12, PROPOSED)
For discussion user selects server-approved business-effective month with immutable current-period adjustment after close. Date used to select historical price version and enterprise contract is still OPEN. A late event must record original usage period, current posting period and explicit correction reason; no silent re-rating. See [Billing §21](../modules/BILLING_ARCHITECTURE.md).
