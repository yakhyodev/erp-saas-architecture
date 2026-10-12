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

## Round 2 clarification — AR-001 / MR-01 (2026-10-12, PROPOSED)
MVP **option B**: Tenant 1:N Company candidate, Company 1:1 BillingAccount candidate, BillingAccount 1:1 Wallet candidate; Company↔LegalEntity relationship and legal payer/debtor explicit, not guessed. Alternatives: A tenant consolidated account, C contract pooling with effective-dated many-company mapping, D branch accounts. Compare [Billing §20.1](../modules/BILLING_ARCHITECTURE.md). **Legal payer funding an account is not necessarily the legal debtor.** Ledger stores source company/branch, liability BillingAccount, legal debtor, payer, wallet, mapping version independently; preserve historic assignments after company transfer. Risk: shared corporate payments, misattributed debts and fiscal treatment. Joint Multi-Tenant/Finance/Business approval OPEN. Earlier single-company wallet wording only a candidate, not APPROVED.

## User-selected candidate follow-up (2026-10-12, PROPOSED)
Company→BillingAccount→Wallet 1:1 MVP option B is **SELECTED_PENDING_APPROVAL**, not APPROVED. Compare pooled tenant, company and enterprise account alternatives in [Billing §20.1/§21](../modules/BILLING_ARCHITECTURE.md). LegalEntity↔Company, account owner/legal debtor and third-party payer funding are not conflated. Source company/branch attribution vs legal-liability ledger attribution must be independently snapshotted. Shared enterprise wallet and account migration remain OPEN. Finance/Multi-Tenant/business approval required.
