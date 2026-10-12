# ADR-BILL-011 — MVP Company-scoped SaaS Billing va Tenant-shared Resource Usage

- **ADR status:** `PROPOSED` — quyidagi belgilangan biznes subqarorlari `APPROVED_BY_USER`; xavfsizlik/texnik mexanizmlar va noaniq edge-case'lar `PROPOSED/OPEN`.
- **Sana:** 2026-10-12
- **Owner:** 01 Billing; bog'liq: 00 Master, 02 Multi-Tenant, 04 Backend, 06 Finance, 08 UX, 10 Security, 11 Infrastructure
- **Issue:** AR-001; MR-01, MR-03, MR-04, MR-11
- **Provenance:** Foydalanuvchining 2026-10-12 dagi ushbu Project suhbatidagi ketma-ket bevosita A-variant tanlovlari, undan keyingi tavsiya qilingan yo'llarni davom ettirish vakolati. Blanket vakolat hech qanday narx/soliq/limit/ilgari ochiq tech ADRni avtomatik APPROVED qilmaydi.
- **Review baseline:** Round 3 audit, `34fbb99db8c71a01dbb50ef71a6f7555a3310e91`.

## 1. Muammo

Bir Tenant bir nechta mustaqil Company va Branchni boshqaradi. SaaS charge manbasi, legally liable debtor, BillingAccount/Wallet va real funding payer bir-biriga tenglashtirilmasin. Tenant-shared, bevosita Company'ga bog'lanmaydigan real Resource Usage uchun aniq javobgar topilishi kerak.

## 2. Ko'rilgan variantlar va tanlov sababi

- **A: Company-scoped account + Tenant shared-resursga bitta mas'ul Company — biznes uchun TANLANGAN.** Soddaroq hisob, izolyatsiya, audit va samarali MVP; umumiy resurs ichki beneficiary xarajatini taqsimlash kerak bo'lsa reportingda alohida ko'riladi.
- **B: Shared-resursni har Companyning aniq o'lchangan ulushi bo'yicha bo'lish — tanlanmagan MVP muqobil.** Qo'shimcha meter/provenance va split-charge murakkabligi.
- **C: Oldindan belgilangan foiz bo'yicha bo'lish — tanlanmagan MVP muqobil.** Adolat/haqiqiy iste'mol farqi, versiyalash, dispute xavfi.
- Tenant-pooled Wallet, branch wallet va enterprise N:M consolidated payer ham MVP uchun tanlanmagan; kelajakda yangi ADR/contract talab qiladi.

## 3. APPROVED_BY_USER — shu suhbatdagi tor biznes qarorlar

1. Bitta Tenant bir nechta Companyga ega bo'lishi mumkin. MVPda **Company:LegalEntity:BillingAccount:Wallet = 1:1:1:1**, har Companyga alohida yuridik shaxs va alohida SaaS hisob/balans. Filialga alohida Wallet yaratilmaydi.
2. **Legal debtor** shu Companyning LegalEntity'si. Boshqa shaxs tegishli vakolat bilan o'sha Company Wallet'iga to'lov qilishi mumkin, lekin oddiy top-up qarzdorlik egasini almashtirmaydi.
3. **Seat = user × Company**: ayni userning bir Company ichidagi bir nechta Branch yoki device bilan ishlashi qo'shimcha Seat emas; boshqa Companyda mustaqil Seat bo'ladi. Aktivatsiya, suspension, kunlik proration, exact cycle va tariflar alohida MR-03 qarori sifatida OPEN.
4. Company/Branch-scoped usage shu Companyga tegishli; haqiqiy o'lchangan **Tenant-shared Resource Usage** uchun **oldindan belgilangan bitta mas'ul Company**ning BillingAccount/Wallet hisobga olinadi.
5. Yuqoridagilar avvalgi Hybrid Prepaid, arzon Seat+Business Usage+Resource Usage, hujjat base+overage, 1 kredit=10 so'm, progressive chegirma, haqiqiy Seat/Usage xarajatini **bir kalendar oy** minusga yozish va **taxminiy kredit limitisiz** ishlash invariantlarini o'zgartirmaydi.

**Qarzdor != payer/funder != beneficiary != source Company.** ERP Finance GL/AR/AP va SaaS Wallet/credit ledger bir xil emas.

## 4. Domen va authoritative ownership kontrakti — PROPOSED

| Fact/object | Writer/validator (taklif) | Kontrakt |
|---|---|---|
| Tenant, Company, Branch va LegalEntity bog'liqligi | Multi-Tenant org register; Finance legal evidence validation | Tenant-scoped, effective-dated, server verified; legal identity change controls OPEN |
| BillingAccount/Wallet, SaaS debt ledger, payer mapping va rated charge | Billing | Distinct per Company, immutable posting/snapshots |
| Resurs iste'moli (fact) | Resource metering source owner | Quantity/unit/window/reference; source narx hisoblamaydi |
| Tenant-shared responsible Company assignment | Billing financial assignment owner; Multi-Tenant verifies Company, Security verifies permissions | Exactly one accepted responsible Company at a given effective interval |
| SaaS liable LegalEntity | Company legally linked entity + Billing contract snapshot | Immutable at charge time, don't infer ERP invoice debtor |
| Third-party top-up | Authorized payment/settlement verifier + Billing | Explicit destination Wallet, funder and evidence; no liability transfer |
| Membership/Seat status fact | Multi-Tenant membership (owner recommendation), Billing rates Seat | Billable state/cadence OPEN; per user/Company identity APPROVED |
| ERP GL/AR/AP | Finance | Never write SaaS Wallet directly |

**Attribution matrix:** Branch resource → owning Company wallet with branch analytic dimension; Company resource → same Company's wallet; Tenant-shared resource → preaccepted responsible Company's wallet; per Company Seat → that Company's wallet, dedup branches/devices.

**Proposed immutable envelope:** `tenant_id`, `source_company_id?`, `source_branch_id?`, `resource_scope`, `resource_id`, `beneficiary_company_ids?`, `usage_occurrence_id`, `event_id`, `metering_interval_start/end`, `quantity`, `unit`, `responsible_company_id`, `assignment_version`, `assignment_effective_from/to`, `billing_account_id`, `wallet_id`, `liable_legal_entity_id`, `payer_party_id?`, `payer_authorization_id?`, `pricing_version_id`, `charge_id?`, `posting_period`, `correlation_id`, `reversal_of_charge_id?`. Exact API/DB schema needs technical sign-off; no client-provided Wallet routing authority.

## 5. Tavsiya etilgan assignment va transfer workflow — PROPOSED

- Tenant administrator mas'ul Company'ni **PROPOSE** qiladi; mas'ul bo'ladigan Company's vakolatli moliyaviy vakili **ACCEPT** qiladi. Security ikkala roldagi scope va revoke/versionni tekshiradi. Bir inson ikki rolni bajara olishi, delegatsiya, timeout va dispute **OPEN**.
- Billing faqat faol Tenant/Company uchun accepted versioned assignment'ni ishlatadi. Bir vaqtda bitta Tenant-shared resource uchun ikki active responsible Company bo'lishi mumkin emas.
- **Future-effective (prospective)**: Company A → Company B o'tishi orqaga ta'sir qilmaydi, eski debtor, Wallet balans, top-up, posted charge/invoice/ledger o'zgarmaydi. A qarzi yoki kredit B'ga avtomatik o'tmaydi; legal novation/transfer alohida kelishiladi.
- **Timing proposal:** ikkala approvaldan keyingi belgilangan **to'liq metering intervali** boshidan kuchga kirishi; backdate yo'q. Interval uzunligi, time-zone, event-based samples, cutover oralig'ida ikki segment aniq o'lchanishi, kechikkan usage va yopilgan oy iqtisodiy ta'siri **OPEN**.
- Yangi pullik shared resource uchun accepted mas'ul Company tayinlanmaguncha charging-enabled bo'lmasin (PROPOSED). Mavjud resource uchun noaniq mapping bo'lsa actual meter fact `PENDING_ATTRIBUTION` bo'lib qolsin; hech qaysi Wallet tanlanmasin, retro-charge faqat kelgusida tasdiqlangan correction policy bilan.
- Old published amount tuzatilishi kerak bo'lsa `reversal_of_charge_id` yoki compensating ledger posting; UPDATE/DELETE emas. Semantic occurrence duplicate replay ikkinchi debit bermasin; same semantic ID + boshqa payload conflict.

## 6. Payer va payment settlement — PROPOSED security controls

Authorized third-party payer `payer_party_id` bilan, belgilangan target `billing_account_id` va `wallet_id`ga to'laydi; payment authorization va provider settlement haqiqiy tekshirilganidan keyin Billing bir marta credit qiladi. Bu source Company qarzini, ERP AR debtorini yoki Billing legal liabilityni qayta belgilamaydi. Wrong-wallet, refund, chargeback va disputes alohida OPEN protsedura; pooling/silent cross-wallet offset yo'q.

## 7. Acceptance matrix (design only, NOT TESTED)

- Tenant A'dagi Company A/B uchun alohida Wallet/LegalEntity, Branch 1/2 va bir userning ikki Company bo'ylab Seat scope'i; branch/device duplicates yo'q.
- Metered Company/Branch resource only own Wallet; Tenant-shared only accepted responsible Company; foreign tenant/forged wallet/revoked role mapping rejected.
- Proposed→accepted→effective boundary, duplicate acceptance, stale version, effective-date replay, late event and closed accounting period; old charge ledger immutable.
- Billing economic occurrence dedup across transport replay; third-party top-up only verified; no estimate-based negative ceiling, no unapproved post-expiry suspension.
- Reconciliation source measured facts ↔ responsible assignment version ↔ liable legal debtor ↔ posted charge, no automatic guessed wallet or cross-company debt transfer.

## 8. Risk va boshqa modullarga ta'sir

Risks: authorized delegate fraud, ambiguous Company assignment, split/window meter disagreement, historical debtor corruption, wrong-wallet funding/refund, duplicate seat/usage, stale cross-tenant cache. Billing, Multi-Tenant, Finance, Security, Backend, Infrastructure, UX va Master reyestrlari ta'sirlanadi. RLS/negative security, recovery va economic replay tests faqat specification — hali bajarilmagan.

## 9. OPEN va governance

- Metering period/timezone, shared-resurs tur va narxi, free threshold, period boundary, delayed/offline attribution; Seat lifecycle/proration/tarif va chargeable hujjat siyosati.
- Dual approval same-person/role policy, exact assignment cutoff/expiration, misattribution and PENDING re-rating, third-party remitter authorization proof, refund/chargeback, legal/tax invoice va yuridik qayta tashkil etish.
- ADRning **umumiy statusi PROPOSED**, chunki ochiq texnik/operatsion kontraktlar bor. Ushbu ADRdagi **aniq biznes tanlovlar APPROVED_BY_USER**. Oldingi ADR-BILL-001/010 tarixiy matnini o'chirish yoki FIN-001 approval provenance'ni taxminan yopish mumkin emas.
