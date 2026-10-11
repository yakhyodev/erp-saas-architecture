# ADR-BILL-008 — Immutable billing invoice aggregation

- **Status:** PROPOSED
- **Date:** 2026-10-11
- **Scope:** Billing Architecture
- **Business-rule authority:** APPROVED biznes invariantlari [Billing module](../modules/BILLING_ARCHITECTURE.md) da; bu ADR texnik usulni tasdiqlamaydi.

## Muammo
SaaS billing invoice va ERP savdo invoice'i ajratilishi, posted charges qayta yozilmasligi kerak.

## Ko'rilgan variantlar
Live mutable bill; immutable finalized billing invoice + credit notes.

## Tanlangan yechim (taklif)
PROPOSED: finalized invoice lines reference immutable charges and price versions; corrections via separate adjustments.

## Sabab
Audit, reconciliation va ERP Finance bilan toza chegaralar.

## Xavflar va ochiq masalalar
Tax legal requirements, numbering, issuance timing OPEN.

## Boshqa modullarga ta'siri
Finance, Payments, Reporting, Localization.

## Qaror tartibi
Foydalanuvchi alohida tasdiqlamaguncha PROPOSED. O'zgartirilsa eski ADR SUPERSEDED sifatida tarixda qoladi.
