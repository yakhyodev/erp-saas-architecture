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

## AR-001 / MR-01 clarification (PROPOSED, not approved)
See [Billing §19.1](../modules/BILLING_ARCHITECTURE.md#191-ar-001--mr-01--tenant--company--billingaccount--wallet). Previous “one company wallet” selection is explicitly **candidate B only**, not cardinality approval. Options: tenant consolidated account (N companies→1 account→1 wallet); company account (1 company→1 billing account→1 wallet); effective-dated flexible contract accounts (many company mappings; 1:N wallet possible); branch wallet candidate. Source company/branch attribution must be separated from account payer/liability, both frozen in each posted entry. No assumption BillingAccount=Tenant or Company=Tenant. Multi-Tenant/Finance joint approval OPEN; changes via linked reversals, never ledger mutation.
