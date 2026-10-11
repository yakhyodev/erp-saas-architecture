# ADR-BILL-002 — Immutable double-entry credit ledger

- **Status:** PROPOSED
- **Date:** 2026-10-11
- **Scope:** Billing Architecture
- **Business-rule authority:** APPROVED biznes invariantlari [Billing module](../modules/BILLING_ARCHITECTURE.md) da; bu ADR texnik usulni tasdiqlamaydi.

## Muammo
Audit, reversal va parallel debitda balans qanday ishonchli yuritiladi?

## Ko'rilgan variantlar
Mutable balance-only; append-only single-entry; append-only double-entry.

## Tanlangan yechim (taklif)
PROPOSED: muvozanatli double-entry posted journal, append-only, linked reversal; wallet balance ledger projection.

## Sabab
Moliyaviy izchillik, reconciliation va tarixni qayta tiklash.

## Xavflar va ochiq masalalar
Murakkabroq posting, contention va projection drift; invariant/monitor talab.

## Boshqa modullarga ta'siri
Database, Finance, Payments, Audit, Security.

## Qaror tartibi
Foydalanuvchi alohida tasdiqlamaguncha PROPOSED. O'zgartirilsa eski ADR SUPERSEDED sifatida tarixda qoladi.
