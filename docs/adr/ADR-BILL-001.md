# ADR-BILL-001 — Tenant-level wallet

- **Status:** PROPOSED
- **Date:** 2026-10-11
- **Scope:** Billing Architecture
- **Business-rule authority:** APPROVED biznes invariantlari [Billing module](../modules/BILLING_ARCHITECTURE.md) da; bu ADR texnik usulni tasdiqlamaydi.

## Muammo
Kompaniya va filial sarfini birlashtirib qanday hisoblash kerak?

## Ko'rilgan variantlar
Alohida branch wallet; kompaniya wallet; enterprise uchun alohida hisoblar.

## Tanlangan yechim (taklif)
PROPOSED: kompaniya billing account uchun yagona wallet, branch attribution ledger entry darajasida.

## Sabab
To'lov va balans boshqaruvi soddalashadi; filial sarfi kesimda ko'rinadi.

## Xavflar va ochiq masalalar
Filial byudjetlari bilan kompaniya wallet mosligi; migratsiyada balans o'zgarishi.

## Boshqa modullarga ta'siri
Tenant, Branch, IAM, Reporting, Finance.

## Qaror tartibi
Foydalanuvchi alohida tasdiqlamaguncha PROPOSED. O'zgartirilsa eski ADR SUPERSEDED sifatida tarixda qoladi.
