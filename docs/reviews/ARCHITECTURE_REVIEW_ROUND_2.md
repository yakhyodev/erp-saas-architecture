# ERP SaaS — INDEPENDENT ARCHITECTURE REVIEW, ROUND 2

> **Holat:** REVIEWED — CORRECTIONS REQUIRED  
> **Loyiha / auditor:** Sales doc copy · 14 — Architecture Review  
> **Audit qilingan branch:** mavjud `main`  
> **CURRENT AUDITED HEAD (frozen evidence snapshot):** [184b32f7b3dcc9ce8d329ca2dfb7895f06cae905](https://github.com/yakhyodev/erp-saas-architecture/tree/184b32f7b3dcc9ce8d329ca2dfb7895f06cae905)  
> **Historical baseline:** [016573c8426ed3ea3b1eacbd0dc81adb67861220](https://github.com/yakhyodev/erp-saas-architecture/tree/016573c8426ed3ea3b1eacbd0dc81adb67861220)  
> **Historical first audit:** [ARCHITECTURE_REVIEW.md](ARCHITECTURE_REVIEW.md)  
> **Review date:** 2026-10-12  
> **Implementation verdict:** **BLOCKED**. This review does not approve technical designs or authorize product code.  
> **Snapshot caveat:** audited source HEAD remains `184b32f7b3dcc9ce8d329ca2dfb7895f06cae905`; this editorial update advances main but does not retroactively change audit evidence.
> **Publication baseline main HEAD:** [776d5190b696c776ebaa2d0a0549daac9b32873b](https://github.com/yakhyodev/erp-saas-architecture/commit/776d5190b696c776ebaa2d0a0549daac9b32873b) (re-checked before editing; previous Round 2 publication).

## 1. Executive Summary

GitHub connector orqali **main HEAD qaytadan tekshirildi**, uning recursive tree’i hamda **barcha 17 modul Markdowni, 97 individual ADR, beshta markaziy registry, README va avvalgi 600 qatorli audit** mazmuni tekshirildi. **121 Markdown = 97 ADR + 17 module docs + 5 root registries + README + first review**; 14 arxitektura yo‘nalishi saqlangan. Tarixiy baseline 108 Markdown/90 ADR/17 module docs edi. Current main baseline’dan **59 commit oldinda**, 59 ta fayl qo‘shilgan yoki o‘zgargan.

**Haqiqiy taraqqiyot:** ilgari yo‘q bo‘lgan ikki Mobile ADR va beshta registry paydo bo‘lgan (2 eski missing-file/document-existence muammosi tuzatilgan); Catalog/Party/tenant va Security ownership takliflari oydinlashgan; Database/Infrastructure bir xil staged HA taklifga kelgan; Sales–Inventory–Finance typed lifecycle matrix yaratilgan; Billing uchun negative-month, seat, rating, document lifecycle variantlari tafsilotli; UX persona × **20 workflow** sinov matritsasi bor; SalesDoc **DOCUMENTED** vs **LIVE_VERIFIED** taxonomy yaratilgan.

**Haqiqiy qolgan muammolar:** hech bir yangi texnik model user-approved emas; BillingAccount/Wallet/legal payer cardinality, finance tax/GL recognition, document charge trigger, late offline month, permission owner, Catalog/Party scope, SLA/RPO/RTO va UX acceptance approval ochiq. Database §6 dagi “SalesDocument + stock + finance journal bitta transaction” ifodasi §17 dagi distinct-domain outbox yondashuvi bilan **bir faylda hozir ham semantik jihatdan muvofiqlashtirilishi zarur**. `MODULE_REGISTRY.md` Securityga membership responsibilityni beradi, Security/Multi-Tenant appendixlari esa TenantMembership writer uchun Multi-Tenantni nomzod qiladi. `OPEN_QUESTIONS.md` AR-006/007ni hamon “missing” deydi — bu **stale tracking**, foydalanuvchining yangi arxitektura tasdig‘i emas.

**Round 2 oldingi 13 ta topilma statuslari:** quyida har biri dalilli va bir necha statusli ifodalangan. **FIXED_DOCUMENTATION (physical gap solved): 2/13 — AR-006 va AR-007**, lekin root registriesdagi tracking hali yangilanishi kerak. **Qolgan 11/13** bo‘yicha kamida user/domain approval, live evidence yoki final contract sign-off talab qilinadi; bu 11ning hammasi “user approval”ga bir xil darajada muhtoj degani emas. **APPROVED-vs-APPROVED ziddiyat:** aniqlanmadi. **Codex:** 12 BLOCKED, 2 READY_WITH_CONDITIONS, 0 READY; umumiy **BLOCKED**.

**Terminlar:** `FIXED_DOCUMENTATION` — eski hujjat/fayl kamchiligi hozir bartaraf etilgan; `HARMONIZED_PROPOSED` — mos va batafsil taklif yozilgan, lekin qaror tasdiqlanmagan; `STILL_CONFLICTING` — joriy matnlar orasida hali o‘zaro farqli authority/transaction ifodasi bor; `MISSING_APPROVAL` — user/domain formal sign-off yoki original provenance yo‘q. Bu belgilar bir topilmada birga kelishi mumkin. **PROPOSED mismatch ≠ APPROVED business conflict**.

---

## 2. Audit Scope & Methodology

### Audit doirasi

- GitHubdagi Markdown manbalar va individual ADRlar audit qilindi; ayrim module hujjatlari tarixiy qismlar va keyingi clarification appendixlaridan iborat.
- No executable application, SQL migration, authenticated SalesDoc production API, RLS/connection-pool test, measured UX/usability/performance, backup restore, failover drill yoki security audit ishga tushirilmadi.
- Historical review faktlari yangisi sifatida ko‘chirilmaydi; Round 2 topilmalari current HEADdagi tegishli bo‘limlar va ADRlarga tayangan.
- Manba nisbiy linklari ushbu `docs/reviews` faylidan ishlaydi; ularning content snapshoti yuqoridagi audited commit SHAga tegishli.
- Severity riskning potensial ta’sirini bildiradi, ishlayotgan tizimdagi tasdiqlangan defekt degani emas.

---

**Publication check:** joriy `main` shu qayta rasmiylashtirishdan avval [776d5190b696c776ebaa2d0a0549daac9b32873b](https://github.com/yakhyodev/erp-saas-architecture/commit/776d5190b696c776ebaa2d0a0549daac9b32873b) edi. Mavjud Round 2 audit ma’lumotlari o‘zgarmagan; yangi vazifa — ayni hisobotni qayta strukturalash. 97 ta ADR / 17 modul bo‘yicha yangi runtime verification o‘tkazilgani da’vo qilinmaydi. Auditni o‘tkazishda GitHub connector orqali joriy manba kontentlari ko‘rilgan, production tizim tekshirilmagan.

---

## 3. Repository Baseline va commit SHA

| Revision | Commit SHA | Ma’nosi |
|---|---|---|
| Historical baseline | [`016573c8426ed3ea3b1eacbd0dc81adb67861220`](https://github.com/yakhyodev/erp-saas-architecture/commit/016573c8426ed3ea3b1eacbd0dc81adb67861220) | Birinchi auditdan oldingi tarixiy arxitektura snapshoti: 108 Markdown / 90 ADR. |
| **Round 2 audited source HEAD** | [`184b32f7b3dcc9ce8d329ca2dfb7895f06cae905`](https://github.com/yakhyodev/erp-saas-architecture/commit/184b32f7b3dcc9ce8d329ca2dfb7895f06cae905) | 00–13 reconciliation hujjatlari bo‘yicha **asosiy dalil snapshoti**: 121 Markdown / 97 ADR / 17 modul. |
| Previous Round 2 report commit | [`776d5190b696c776ebaa2d0a0549daac9b32873b`](https://github.com/yakhyodev/erp-saas-architecture/commit/776d5190b696c776ebaa2d0a0549daac9b32873b) | Mazkur review’ning avvalgi 12-bo‘limli to‘liq versiyasi, review faylining tarixida qoladi. |
| Main HEAD before this editorial update | `776d5190b696c776ebaa2d0a0549daac9b32873b` | 23-bo‘limga qayta tartiblashdan avval GitHub’dan tasdiqlandi; publication commit GitHub commit history va yakuniy javobda beriladi. |

**Hisoblash farqi:** Round 2 review ilk bor qo‘shilgach joriy repositorydagi Markdown soni 122 bo‘lgan; auditning o‘zi **121** hujjatli oldingi HEADga asoslangan. Shu ikki inventarni aralashtirmang. [Historical review](ARCHITECTURE_REVIEW.md) o‘zgartirilmaydi.

---

## 4. Markdown Documents & ADR Inventory

| Kategoriya | Baseline | Current HEAD | Dalil |
|---|---:|---:|---|
| Barcha Markdown | 108 | **121** | Recursive `git/trees/main?recursive=1`, audited SHA |
| 00–13 arxitektura yo‘nalishlari | 14 | **14** | [Module Registry](../../MODULE_REGISTRY.md) va modul fayllari |
| `docs/modules/` | 17 | **17** | [modullar](../modules/) |
| `docs/adr/` | 90 | **97** | [ADRlar](../adr/) |
| Root registries | 0 | **5** | [DECISIONS](../../DECISIONS.md), [REQUIREMENTS](../../REQUIREMENTS.md), [MODULE_REGISTRY](../../MODULE_REGISTRY.md), [OPEN_QUESTIONS](../../OPEN_QUESTIONS.md), [CHANGELOG](../../CHANGELOG.md) |
| README | 1 | **1** | [README](../../README.md) |
| Birinchi audit | Baseline’dan so‘ng | **1** | [ARCHITECTURE_REVIEW](ARCHITECTURE_REVIEW.md) |
| Missing Mobile ADR 004/009 | 2 missing | **0 missing** | [ADR-MOB-004](../adr/ADR-MOB-004.md), [ADR-MOB-009](../adr/ADR-MOB-009.md) |

### 2.1 Hujjatlarning real joylashuvi / qamrovi

| Chat | Current primary/module docs | ADR oilasi |
|---|---|---|
| 00 Master | [MASTER_ARCHITECTURE.md](../modules/MASTER_ARCHITECTURE.md) | ADR-001…009 (9) |
| 01 Billing | [BILLING_ARCHITECTURE.md](../modules/BILLING_ARCHITECTURE.md) | ADR-BILL-001…010 (10) |
| 02 Multi-Tenant | [MULTI_TENANT_ARCHITECTURE.md](../modules/MULTI_TENANT_ARCHITECTURE.md) | ADR-MT-001…002 (2) |
| 03 Database | [DATABASE_ARCHITECTURE.md](../modules/DATABASE_ARCHITECTURE.md) | ADR-DB-001…009 (9) |
| 04 Backend | [BACKEND_ARCHITECTURE.md](../modules/BACKEND_ARCHITECTURE.md) | ADR-BE-001…008 (8) |
| 05 Sales/Inventory | [SALES_INVENTORY.md](../modules/SALES_INVENTORY.md) | ADR-SI-001…013 (13) |
| 06 Finance | [FINANCE_ARCHITECTURE.md](../modules/FINANCE_ARCHITECTURE.md), [FINANCE_ACCOUNTING.md](../modules/FINANCE_ACCOUNTING.md) | FIN-001…002 (2) |
| 07 CRM | [CRM_DISTRIBUTION.md](../modules/CRM_DISTRIBUTION.md) | ADR-CRM-001…005 (5) |
| 08 UX/UI | [UX_ARCHITECTURE.md](../modules/UX_ARCHITECTURE.md), [UX_UI_ARCHITECTURE.md](../modules/UX_UI_ARCHITECTURE.md) | ADR-UX-001…006 (6) |
| 09 Mobile | [MOBILE_ARCHITECTURE.md](../modules/MOBILE_ARCHITECTURE.md), [MOBILE_OFFLINE.md](../modules/MOBILE_OFFLINE.md) | ADR-MOB-001…009 (9) |
| 10 Security | [SECURITY_ARCHITECTURE.md](../modules/SECURITY_ARCHITECTURE.md) | ADR-SEC-001…010 (10) |
| 11 Infra | [INFRA_ARCHITECTURE.md](../modules/INFRA_ARCHITECTURE.md) | ADR-INFRA-001…005 (5) |
| 12 Integrations | [INTEGRATION_ARCHITECTURE.md](../modules/INTEGRATION_ARCHITECTURE.md) | ADR-INT-001…005 (5) |
| 13 Localization | [LOCALIZATION_ARCHITECTURE.md](../modules/LOCALIZATION_ARCHITECTURE.md) | ADR-LOC-001…004 (4) |

**97 ta ADRni status + muammo + variant + tanlov + risk + ta’sir bo‘yicha tekshirish:** 96 **PROPOSED**; bittasida [FIN-001](../adr/FIN-001.md) **APPROVED** yozilgan, lekin explicit foydalanuvchi approval provenance **UNVERIFIED**. FIN-001 statusi o‘zgartirilmaydi. Raqamli statusning o‘zi user authorizationni isbotlamaydi. ADRlarning aksariyati qisqa, variant va riskni nomlaydi, ammo executable acceptance contract emas: ayniqsa role matrix, event schema catalog, pricing/legal policy, perf/DR numeric objective va test evidence hali missing. [DECISIONS](../../DECISIONS.md) atigi TECH-001..008 va biznes reyestrini qisqacha beradi; **97 ADR uchun to‘liq status/provenance index emas**.

**Historical descriptions:** [UX_UI_ARCHITECTURE](../modules/UX_UI_ARCHITECTURE.md) o‘zining ADR-UX-001…003 eski konsept yorliqlari canonical emasligini to‘g‘ri eslatadi; canonical [UX §12](../modules/UX_ARCHITECTURE.md). [MASTER_ARCHITECTURE](../modules/MASTER_ARCHITECTURE.md) hali **PROPOSED modul hujjati**, mustaqil root APPROVED master emas. [README](../../README.md) arxitektura planning deydi. Root registrylar yaratilganiga qaramay ularning ayrim jadvali tarixiy AR holatini avtomatik stale holda saqlagan.

---

---

## 5. APPROVED Business Rules Validation

| ID | User-approved invariant | Current document evidence | Status |
|---|---|---|---|
| BIZ-001 | Hybrid Prepaid SaaS | [DECISIONS — APPROVED](../../DECISIONS.md), [Billing §1](../modules/BILLING_ARCHITECTURE.md) | Mos |
| BIZ-002 | Arzon Seat + Business Usage + Resource Usage | [REQUIREMENTS — APPROVED](../../REQUIREMENTS.md), [Billing §6–8](../modules/BILLING_ARCHITECTURE.md) | Mos |
| BIZ-003 | Hujjat asosiy narxi + threshold oshgach qo‘shimcha kredit | [Billing §7, §19.2](../modules/BILLING_ARCHITECTURE.md), [ADR-BILL-009](../adr/ADR-BILL-009.md) | Mos; eligible hujjat turi va trigger OPEN |
| BIZ-004 | **1 kredit = 10 so‘m** | [Billing §1](../modules/BILLING_ARCHITECTURE.md), [Database §4.5](../modules/DATABASE_ARCHITECTURE.md) | Mos |
| BIZ-005 | Progressive discount + enterprise contract | [Billing §9–10, §19.6](../modules/BILLING_ARCHITECTURE.md) | Mos; marginal vs retroactive tier/precedence OPEN |
| BIZ-006 | Nol/minusdan so‘ng **haqiqiy** Seat va Usage xarajatlari **bir kalendar oy** minusda | [ADR-BILL-006](../adr/ADR-BILL-006.md), [Billing §19.4](../modules/BILLING_ARCHITECTURE.md) | Mos; month anchor/after-expiry OPEN |
| BIZ-007 | **Taxminiy minus kredit limiti yo‘q** | [Billing §13, §19.4](../modules/BILLING_ARCHITECTURE.md), [Master §8](../modules/MASTER_ARCHITECTURE.md) | Mos; forecast ceiling tasdiqlanmagan va kiritilmaydi |
| BIZ-008 | Kompaniya/filiallar yagona SaaSda | [Multi-Tenant §1/§15](../modules/MULTI_TENANT_ARCHITECTURE.md) | Mos; Tenant=Company emas |
| BIZ-009 | uz/en/ru + yashirin yangi o‘zbek alifbosi | [Localization §1, §7, §15](../modules/LOCALIZATION_ARCHITECTURE.md) | Mos |
| BIZ-010 | Sales Doctor’dan **tushunarliroq UX** ustuvorligi | [UX §1, §16](../modules/UX_ARCHITECTURE.md) | Maqsad qayd etilgan, real superiority measure yo‘q |

**Noto‘g‘ri status bermaslik:** [DECISIONS TECH-001…008](../../DECISIONS.md) bo‘yicha Go, PostgreSQL, Next.js, Flutter, modular monolith, shared DB/RLS, sharding, dedicated DB, REST/OpenAPI, queue/HA va i18n internal locale identifiers — **PROPOSED**. Hali code release, performance/security acceptance yoki oldingi qonuniy FIN-001 tasdig‘i isbotlanmagan. Hozirgi review ularni APPROVED qilmaydi.

**FIN-001 exception:** [FIN-001](../adr/FIN-001.md) va [Finance addendum](../modules/FINANCE_ARCHITECTURE.md) ledger boundary ajratilishini foydali, texnik jihatdan mantiqli deb ko‘rsatadi, ammo original explicit approval manbasi berilmagan; [DECISIONS — Historical provenance exception](../../DECISIONS.md) ayni kamchilikni tan oladi. **Status qo‘l tekkizilmasin; provenance yechilsin.**

---

---

## 6. AR-001 — AR-013 Findings Matrix

Bir AR uchun bir necha holat berilishi mumkin. “Harmonized” user-approved degani emas. **Severity Round 2 design risks**, historical severity avtomatik ko‘chirilmadi.

| ID | Severity | Round 2 classification | Joriy holat va tekshirilgan manbalar | Qolgan qaror / egasi |
|---|---|---|---|---|
| **AR-001** | HIGH | **HARMONIZED_PROPOSED + MISSING_APPROVAL** | [Billing §19.1](../modules/BILLING_ARCHITECTURE.md) oldingi “company wallet”ni option B deb chekladi; [Multi-Tenant §15](../modules/MULTI_TENANT_ARCHITECTURE.md), [ADR-BILL-001](../adr/ADR-BILL-001.md) Tenant/Company/LegalEntity/BillingAccount/Wallet, source vs liability attributionni ajratadi. | 1:1/1:N payer/wallet/seat allocation: 01+02+06+user. |
| **AR-002** | HIGH | **HARMONIZED_PROPOSED + MISSING_APPROVAL** | [CRM ADR-001](../adr/ADR-CRM-001-domain-ownership.md) oldingi “Sales owns products” umumiy ifodasini to‘g‘riladi; [Sales §11.1](../modules/SALES_INVENTORY.md), [Backend §17](../modules/BACKEND_ARCHITECTURE.md), [ADR-SI-011](../adr/ADR-SI-011.md): Catalog/Product, Pricing, Sales, Inventory, Purchasing writerlari ajratilgan. | Catalog tenant/global scope, implementation container, SKU identity: 00+04+05+07+user. |
| **AR-003** | MEDIUM | **HARMONIZED_PROPOSED + MISSING_APPROVAL** | [Database §9/§17](../modules/DATABASE_ARCHITECTURE.md), [ADR-DB-007](../adr/ADR-DB-007.md), [Infra §7/§18](../modules/INFRA_ARCHITECTURE.md): Stage A single writer+PITR **not HA**, B standby+fencing, C measured automatic failover. Tarixiy takliflar moslashtirilgan. | Initial Stage A/B, RPO/RTO, downtime, budget: 03+11+user. |
| **AR-004** | HIGH | **MISSING_APPROVAL** | [FIN-001](../adr/FIN-001.md), [DECISIONS provenance exception](../../DECISIONS.md), [Finance addendum](../modules/FINANCE_ARCHITECTURE.md): APPROVED yorliq saqlangan, original explicit user approval independently unverified. | Approval source yoki formal status resolution: 06+00+user; ADR statusini bu review o‘zgartirmaydi. |
| **AR-005** | HIGH | **HARMONIZED_PROPOSED + STILL_CONFLICTING + MISSING_APPROVAL** | [Sales §11.2–11.3](../modules/SALES_INVENTORY.md), [Finance reconciliation](../modules/FINANCE_ARCHITECTURE.md), [Database §17](../modules/DATABASE_ARCHITECTURE.md) confirm!=physical stock, dispatch once, invoice no second stock, typed events deydi. Lekin **[Database §6](../modules/DATABASE_ARCHITECTURE.md)** eski “SalesDocument posting + StockMovement + Finance journal in one DB transaction” iborasini hali saqlaydi; **§17** esa source-domain atomic+outbox, cross-domain GL timing OPEN. Bu approval emas, ammo implementer uchun hozir ham ikki talqin. | Canonical transaction boundary, GL/AR/revenue/COGS trigger, typed event policy: 03+04+05+06+user. |
| **AR-006** | MEDIUM | **FIXED_DOCUMENTATION + HARMONIZED_PROPOSED + MISSING_APPROVAL** | [ADR-MOB-004](../adr/ADR-MOB-004.md) va [ADR-MOB-009](../adr/ADR-MOB-009.md) endi haqiqatan mavjud; [Mobile reconciliation](../modules/MOBILE_ARCHITECTURE.md), [Security §18](../modules/SECURITY_ARCHITECTURE.md) queued-command reauthorization, stable operation_id, revoke handlingni moslashtirgan. | Offline TTL, device cleanup, credential lifetime va conflict retention approval: 09+10+04. [OPEN_QUESTIONS](../../OPEN_QUESTIONS.md)dagi “missing ADR” tarixi yangilansin. |
| **AR-007** | MEDIUM | **FIXED_DOCUMENTATION + STILL_CONFLICTING** (stale registry) | [DECISIONS](../../DECISIONS.md), [REQUIREMENTS](../../REQUIREMENTS.md), [MODULE_REGISTRY](../../MODULE_REGISTRY.md), [OPEN_QUESTIONS](../../OPEN_QUESTIONS.md), [CHANGELOG](../../CHANGELOG.md) mavjud; ammo [OPEN_QUESTIONS](../../OPEN_QUESTIONS.md) hali “missing root registries”ni OPEN va [REQUIREMENTS](../../REQUIREMENTS.md) hali eski AR-006/007 matnlarini saqlaydi. Root ADR registry ham 97 ADR bo‘yicha to‘liq emas. | Missing-file statusini resolved/documentedga ajratish, MR registry va provenance update: 00+14. Yangi biznes approval shart emas, lekin har qanday mazmuniy status approval bilan. |
| **AR-008** | HIGH | **HARMONIZED_PROPOSED + MISSING_APPROVAL** | [CRM §11](../modules/CRM_DISTRIBUTION.md), [Finance — identity and settlement](../modules/FINANCE_ACCOUNTING.md), [Integration §AR-008](../modules/INTEGRATION_ARCHITECTURE.md), [ADR-INT-005](../adr/ADR-INT-005.md): Party/Customer/Outlet/Payer/ReceivableOwner farqlangan, CRM vs shared Party canonical writer hali OPEN. | Party writer/merge, legal debtor and third-party payer: 07+06+05+12+user. |
| **AR-009** | HIGH | **HARMONIZED_PROPOSED + MISSING_APPROVAL** | [Billing §19.3–19.4](../modules/BILLING_ARCHITECTURE.md), [Mobile reconciliation](../modules/MOBILE_ARCHITECTURE.md), [ADR-MOB-008](../adr/ADR-MOB-008.md): client time untrusted; server effective/accepted, received, rated, posted va attributed period farqlangan; authoritative period tanlovi OPEN. | Kechikkan event month, close/correction, timezone, negative month anchor: 01+09+04+user. |
| **AR-010** | HIGH | **HARMONIZED_PROPOSED + MISSING_APPROVAL** | [Billing §19.2](../modules/BILLING_ARCHITECTURE.md), [ADR-BILL-009](../adr/ADR-BILL-009.md), [Sales §11.4](../modules/SALES_INVENTORY.md), [Integration §AR-010](../modules/INTEGRATION_ARCHITECTURE.md): document-type × transition × billable occurrence, import lifecycle, dedup va adjustment takliflari mos. | Qaysi hujjat/state chargeable, base fee per logical doc, partial/cancel/return/import refund: 01+05+12+user. |
| **AR-011** | MEDIUM | **HARMONIZED_PROPOSED + MISSING_APPROVAL** (external evidence) | [Integration — Evidence taxonomy](../modules/INTEGRATION_ARCHITECTURE.md) va [CRM §11](../modules/CRM_DISTRIBUTION.md) aynan DOCUMENTED, LIVE_VERIFIED, INFERRED, UNVERIFIED semantikasida kelishgan; [ADR-INT-001](../adr/ADR-INT-001.md). SalesDoc LIVE_VERIFIED emas. | Production/sandbox verification va outbound/cutover ruxsati: 12+07+Security/business. |
| **AR-012** | MEDIUM | **FIXED_DOCUMENTATION (matrix only) + HARMONIZED_PROPOSED + MISSING_APPROVAL** | [UX §16.1](../modules/UX_ARCHITECTURE.md) UX-T01…T20 persona × task × forbidden-outcome matrix; §16.2–16.6 search/context/offline/a11y gate; [ADR-UX-004](../adr/ADR-UX-004.md). **No test passed/approved top-20.** | MVP ordering, role/device coverage, users & quantitative targets: 08+00+all domains+user. |
| **AR-013** | HIGH | **HARMONIZED_PROPOSED + STILL_CONFLICTING + MISSING_APPROVAL** | [Security §17](../modules/SECURITY_ARCHITECTURE.md), [ADR-SEC-002](../adr/ADR-SEC-002.md), [Backend §17](../modules/BACKEND_ARCHITECTURE.md) identity→Multi-Tenant Membership→Security role/grantsni taklif qiladi, **lekin [MODULE_REGISTRY](../../MODULE_REGISTRY.md)** chat 10ga “Identity, membership” vazifasini yozgan; [ADR-MT-002](../adr/ADR-MT-002.md) ayni nomuvofiqlikni qayd etadi; [ADR-BILL-010](../adr/ADR-BILL-010.md) ham “Identity/Security owns memberships” deydi. | **Bitta** TenantMembership writer va role/grant owner, revoke/version policy: 02+10+04+00+user. |

---

## 7. MR-01 — MR-20 Requirements Matrix

**MR status legend:** `SPECIFIED_PROPOSED` (candidate contract va alternativlar yozilgan); `DOCUMENTATION_FIXED` (ilgari yetishmagan qism kiritilgan); `OPEN_BUSINESS` (product/price/legal/role policy user decision); `OPEN_TECH` (ADR/cross-module technical sign-off yoki evidence); `NOT_TESTED` (acceptance dalili yo‘q). Bir nechta maqom normal.

| MR | Round 2 status | GitHub dalil va nima o‘zgardi | Qolgan majburiy qaror |
|---|---|---|---|
| MR-01 Tenant/Company/LegalEntity/BillingAccount | **SPECIFIED_PROPOSED + OPEN_BUSINESS** | [MT §15](../modules/MULTI_TENANT_ARCHITECTURE.md), [Billing §19.1](../modules/BILLING_ARCHITECTURE.md) A/B/C/D mapping variantlari | Payer, debt holder, wallet cardinality, branch responsibility |
| MR-02 Negative calendar month lifecycle | **SPECIFIED_PROPOSED + OPEN_BUSINESS** | [Billing §19.4](../modules/BILLING_ARCHITECTURE.md), [ADR-BILL-006](../adr/ADR-BILL-006.md) T0–T5, named calendar/anniversary variants | “Bir kalendar oy” anchor/timezone/refill/expiry/repeat, no ceiling |
| MR-03 Seat statuses/proration | **SPECIFIED_PROPOSED + OPEN_BUSINESS** | [Billing §19.5](../modules/BILLING_ARCHITECTURE.md), [ADR-BILL-010](../adr/ADR-BILL-010.md) activate/suspend/seat dedup | Billable status/cadence/scope/proration/seat price |
| MR-04 Document & Resource Usage classification | **SPECIFIED_PROPOSED + OPEN_BUSINESS** | [Billing §19.2/§19.6](../modules/BILLING_ARCHITECTURE.md), [ADR-BILL-009](../adr/ADR-BILL-009.md) event matrix | Chargeable types, per-doc unit/threshold, resource types/rates |
| MR-05 Progressive discounts/enterprise | **SPECIFIED_PROPOSED + OPEN_BUSINESS** | [Billing §19.6](../modules/BILLING_ARCHITECTURE.md), [ADR-BILL-007](../adr/ADR-BILL-007.md) marginal vs retroactive | Tier scope/threshold, stacking, enterprise overrides, rounding |
| MR-06 Finance jurisdiction, legal GL | **SPECIFIED_PROPOSED + OPEN_BUSINESS** | [FINANCE_ARCHITECTURE addendum](../modules/FINANCE_ARCHITECTURE.md), [FIN-002](../adr/FIN-002.md) AR/GL source mapping | Jurisdiction, tax point, functional currency, period close/COGS |
| MR-07 Sales/Stock/Finance effects | **SPECIFIED_PROPOSED + OPEN_TECH + OPEN_BUSINESS** | [Sales §11.2–11.3](../modules/SALES_INVENTORY.md), [Database §6/§17](../modules/DATABASE_ARCHITECTURE.md), [ADR-SI-012](../adr/ADR-SI-012.md) | Canonical event transaction owner, Finance recognition trigger |
| MR-08 Stock reservation/valuation/count | **SPECIFIED_PROPOSED + OPEN_BUSINESS** | [Sales §11.5](../modules/SALES_INVENTORY.md), [ADR-SI-004](../adr/ADR-SI-004.md), [ADR-SI-008](../adr/ADR-SI-008.md) | Negative stock, TTL, FIFO/average, count cutoff |
| MR-09 Party/customer/payer master | **SPECIFIED_PROPOSED + OPEN_BUSINESS** | [CRM §11](../modules/CRM_DISTRIBUTION.md), [Finance Accounting](../modules/FINANCE_ACCOUNTING.md) | Party owner, legal identity/merge, third-party payer mapping |
| MR-10 API/event compatibility | **SPECIFIED_PROPOSED + OPEN_TECH** | [Backend §17](../modules/BACKEND_ARCHITECTURE.md), [Integration — MR-10](../modules/INTEGRATION_ARCHITECTURE.md) | Final OpenAPI/event schema registry, retention, version evolution |
| MR-11 Isolation/RLS/placement | **SPECIFIED_PROPOSED + OPEN_TECH + NOT_TESTED** | [Security §19](../modules/SECURITY_ARCHITECTURE.md), [MT ADR-001](../adr/ADR-MT-001.md), [DB §17](../modules/DATABASE_ARCHITECTURE.md) | Choose DB isolation, privileges/pooling, negative RLS tests |
| MR-12 Mobile offline sync/session | **DOCUMENTATION_FIXED + SPECIFIED_PROPOSED + OPEN_TECH** | [ADR-MOB-004](../adr/ADR-MOB-004.md), [ADR-MOB-009](../adr/ADR-MOB-009.md) | Offline TTL, retention, device trust, sync conflict operator policy |
| MR-13 GPS/media/privacy | **SPECIFIED_PROPOSED + OPEN_BUSINESS** | [CRM §4/§6](../modules/CRM_DISTRIBUTION.md), [ADR-CRM-002](../adr/ADR-CRM-002-offline-gps-privacy.md), [ADR-MOB-006](../adr/ADR-MOB-006.md) | Lawful basis, consent, accuracy, retention/erasure |
| MR-14 HA/SLO/RPO/RTO | **SPECIFIED_PROPOSED + OPEN_BUSINESS + NOT_TESTED** | [Infra §18](../modules/INFRA_ARCHITECTURE.md), [ADR-DB-007](../adr/ADR-DB-007.md) | Stage A/B, numeric RPO/RTO, drill, retention, region |
| MR-15 Peak load/performance | **SPECIFIED_PROPOSED + OPEN_TECH + NOT_TESTED** | [Infra §6/§12/§18](../modules/INFRA_ARCHITECTURE.md), [DB §7](../modules/DATABASE_ARCHITECTURE.md) | Real workload envelope, DB connection budget, p95/p99 |
| MR-16 Top-20 UX tasks | **DOCUMENTATION_FIXED + SPECIFIED_PROPOSED + OPEN_BUSINESS + NOT_TESTED** | [UX §16.1](../modules/UX_ARCHITECTURE.md) T01–T20 | Approved prioritization, role/device matrix, pilot evidence |
| MR-17 Global search security | **SPECIFIED_PROPOSED + OPEN_TECH + NOT_TESTED** | [UX §16.3](../modules/UX_ARCHITECTURE.md), [Localization §15](../modules/LOCALIZATION_ARCHITECTURE.md), [ADR-UX-003](../adr/ADR-UX-003.md) | Index owner, search freshness, cache & authorization test |
| MR-18 HR/Manufacturing/Reporting/Purchasing scope | **OPEN_BUSINESS** | [Master §2/§12](../modules/MASTER_ARCHITECTURE.md), [Backend §3](../modules/BACKEND_ARCHITECTURE.md) | MVP inclusion/exclusion, owner, delivery roadmap |
| MR-19 SalesDoc evidence/cutover | **SPECIFIED_PROPOSED + OPEN_TECH + NOT_TESTED** | [Integration taxonomy + cutover](../modules/INTEGRATION_ARCHITECTURE.md), [ADR-INT-003](../adr/ADR-INT-003.md) | Live test, rate limits, support endpoints, signed cutover plan |
| MR-20 A11y/device/network/locale baseline | **SPECIFIED_PROPOSED + OPEN_BUSINESS + NOT_TESTED** | [UX §16.5–16.6](../modules/UX_ARCHITECTURE.md), [Localization §15](../modules/LOCALIZATION_ARCHITECTURE.md) | Test profiles, accepted completion/latency targets |

**Registry gap:** [OPEN_QUESTIONS](../../OPEN_QUESTIONS.md) only AR-001…013 decision packets in a separate section; **MR-01…MR-20ning barchasi uchun mustaqil status/owner/dependency/approval provenance jadvali yo‘q**. Bu hali amalga oshirilmagan hujjat sinxronlashuvidir; ushbu review boshqa faylni o‘zgartirmaydi.

---

---

## 8. Cross-Module Architecture Consistency

| Juftlik yoki zanjir | Current design assessment | Dalil |
|---|---|---|
| Master ↔ All domains | **PARTIAL** — PROPOSED modular monolith/ownership mos, lekin canonical approved master va full ADR registry yo‘q | [Master §15](../modules/MASTER_ARCHITECTURE.md), [MODULE_REGISTRY](../../MODULE_REGISTRY.md) |
| Tenant ↔ Billing ↔ Finance | **HARMONIZED_PROPOSED**, payer/debt cardinality OPEN | [MT §15](../modules/MULTI_TENANT_ARCHITECTURE.md), [Billing §19.1](../modules/BILLING_ARCHITECTURE.md), [Finance addendum](../modules/FINANCE_ARCHITECTURE.md) |
| Backend ↔ Catalog ↔ Sales ↔ Inventory | **HARMONIZED_PROPOSED**, catalog scope not approved | [Backend §17](../modules/BACKEND_ARCHITECTURE.md), [Sales §11.1](../modules/SALES_INVENTORY.md), [CRM ADR-001](../adr/ADR-CRM-001-domain-ownership.md) |
| CRM ↔ Sales ↔ Finance ↔ Integration | **HARMONIZED_PROPOSED**, Party writer/payer unresolved | [CRM §11](../modules/CRM_DISTRIBUTION.md), [Finance Accounting](../modules/FINANCE_ACCOUNTING.md), [INT-005](../adr/ADR-INT-005.md) |
| Sales ↔ Inventory ↔ Finance GL | **STILL_CONFLICTING (transaction description)** with compatible typed lifecycle proposal; trigger OPEN | [DB §6 vs §17](../modules/DATABASE_ARCHITECTURE.md), [Sales §11.2](../modules/SALES_INVENTORY.md), [FIN-002](../adr/FIN-002.md) |
| Database ↔ Security ↔ Tenant | **HARMONIZED_PROPOSED**, conditional shared schema/RLS + composite FK/permission; tests not run | [Security §19](../modules/SECURITY_ARCHITECTURE.md), [DB §5/§17](../modules/DATABASE_ARCHITECTURE.md), [MT ADR-001](../adr/ADR-MT-001.md) |
| Database ↔ Infrastructure | **HARMONIZED_PROPOSED**, single primary PITR not HA, standby later conditional | [DB §17](../modules/DATABASE_ARCHITECTURE.md), [Infra §18](../modules/INFRA_ARCHITECTURE.md) |
| Billing ↔ Mobile ↔ Integrations | **HARMONIZED_PROPOSED**, chargeable month/refund/import not approved | [Billing §19.2–19.4](../modules/BILLING_ARCHITECTURE.md), [Mobile §AR-009](../modules/MOBILE_ARCHITECTURE.md), [Integration §AR-010](../modules/INTEGRATION_ARCHITECTURE.md) |
| Security ↔ Mobile device auth | **HARMONIZED_PROPOSED**, revocation at server reception, wipe offline not guaranteed | [Security §18](../modules/SECURITY_ARCHITECTURE.md), [ADR-MOB-009](../adr/ADR-MOB-009.md) |
| Identity ↔ TenantMembership ↔ Permission grants | **STILL_CONFLICTING ownership registry**, candidate design otherwise aligned | [MODULE_REGISTRY](../../MODULE_REGISTRY.md), [Security §17](../modules/SECURITY_ARCHITECTURE.md), [ADR-MT-002](../adr/ADR-MT-002.md) |
| UX ↔ All business modules | **HARMONIZED_PROPOSED**, testing/MVP approval open | [UX §16](../modules/UX_ARCHITECTURE.md), [ADR-UX-004](../adr/ADR-UX-004.md) |
| Localization ↔ Search ↔ Mobile ↔ Web | **HARMONIZED_PROPOSED**, exact identifiers protected, rollout search test unrun | [Localization §15](../modules/LOCALIZATION_ARCHITECTURE.md), [ADR-LOC-003](../adr/ADR-LOC-003.md), [UX §16.3](../modules/UX_ARCHITECTURE.md) |

---

## 9. Domain Ownership & Contract Conflicts

- **Company/branch context:** Tenant Registry resolves immutable tenant ownership; company legal/accounting identity and Billing payer are **different pending models**. No producer guesses wallet from tenant ID. [MT §15](../modules/MULTI_TENANT_ARCHITECTURE.md), [Billing §19.1](../modules/BILLING_ARCHITECTURE.md).
- **Catalog/Price:** Catalog writer for Product/SKU/UoM, Pricing writer for PriceList; Sales consumes resolved snapshot, Inventory owns physical movement and reservation, Finance owns ERP GL, Billing owns SaaS charge. [Sales §11.1](../modules/SALES_INVENTORY.md), [Backend §17](../modules/BACKEND_ARCHITECTURE.md).
- **Order:** Confirm commercial state; reservation if approved reduces available only; dispatch once decrements physical stock; invoice must **not** cause another stock dispatch; Finance posting is only on explicitly approved accounting trigger. [Sales §11.2](../modules/SALES_INVENTORY.md), [Finance addendum](../modules/FINANCE_ARCHITECTURE.md).
- **Event processing:** versioned typed facts + canonical business occurrence id separate from transport event id; local atomic writes + outbox, consumer inbox/dedup, reversal references original; same-key different payload is conflict. **Single cross-context ACID vs eventual GL** must be definitively chosen and harmonized with DB §6. [ADR-SI-012](../adr/ADR-SI-012.md), [Backend §17](../modules/BACKEND_ARCHITECTURE.md).
- **Billing:** source emits non-monetary transition fact, Billing alone resolves approved eligibility/rates and posts immutable actual charges; offline client never rates/posts. Pricing version, tier, invoice and negative lifecycle OPEN. [ADR-BILL-009](../adr/ADR-BILL-009.md), [ADR-BILL-010](../adr/ADR-BILL-010.md), [ADR-MOB-004](../adr/ADR-MOB-004.md).
- **Security:** identity principal, tenant graph, membership, grants, use-case guard, RLS transaction context independently checked; no authentication token or UI button by itself grants cross-tenant access. [Security §17–19](../modules/SECURITY_ARCHITECTURE.md).

---

**Aniqlanmagan authority ziddiyati:** [`MODULE_REGISTRY.md`](../../MODULE_REGISTRY.md) Security uchun membership mas’uliyatini umumiy tarzda ko‘rsatadi, [Security §17](../modules/SECURITY_ARCHITECTURE.md) va [ADR-MT-002](../adr/ADR-MT-002.md) esa tenant membership state uchun Multi-Tenantni canonical writer sifatida taklif qiladi. [ADR-BILL-010](../adr/ADR-BILL-010.md) “Identity/Security owns memberships” ifodasini ishlatadi. Bu **STILL_CONFLICTING PROPOSED ownership wording**, biror APPROVED qarorni inkor etmaydi. **Owner:** 00/02/04/10/01. **Next:** bitta source-of-truth, role/grant va revocation contractni kelishish.

---

## 10. Billing & Financial Integrity Review

Manbalar: [Billing §19.1–19.7](../modules/BILLING_ARCHITECTURE.md), [Finance addendum](../modules/FINANCE_ARCHITECTURE.md), [FIN-001](../adr/FIN-001.md), [ADR-BILL-002](../adr/ADR-BILL-002.md), [ADR-BILL-003](../adr/ADR-BILL-003.md), [ADR-BILL-006](../adr/ADR-BILL-006.md).

- Billing Wallet/credit ledger **not** tenant ERP cash/GL/AR; FIN-001 provenance unverified.
- Ledger append-only, once-only economic effect, verified payment top-up, price-version snapshots, reversal/adjustment instead of mutation — all technical mechanisms PROPOSED, not tested.
- Core business invariants unchanged: **1 credit=10 so‘m; only actual Seat + Business Usage + Resource Usage; one calendar month negative; no estimated credit ceiling**.
- **STOP before coding:** negative window interpretation, refill/reset/expiry/suspension, chargeable document type/event, imported historical docs, cancellation/refund, late event effective month, seat eligibility/proration, volume thresholds/tier stacking, legal payer mapping. UI must distinguish posted charge from billable candidate/queued sync and invoice from SaaS statement.

---

## 11. Multi-Tenant & Security Review

Manbalar: [SECURITY §17–19](../modules/SECURITY_ARCHITECTURE.md), [ADR-SEC-003](../adr/ADR-SEC-003.md), [ADR-SEC-004](../adr/ADR-SEC-004.md), [ADR-SEC-010](../adr/ADR-SEC-010.md), [MT ADR-001](../adr/ADR-MT-001.md).

**Strong architectural controls proposed:** global IdentityAuth, tenant-scoped membership, Security permission catalog + RBAC/ABAC, default deny, version invalidation, branch/resource guards, composite tenant FK, optional FORCE RLS runtime no bypass, transaction-local pool context, worker/cache/search/file/export namespace, signed webhooks, secrets/KMS, audit/acting-as, short-lived mobile credentials with next-contact device revoke.

**Gate tests NOT EXECUTED:** tenant A→B list/detail/write/lookup/count/autocomplete/search/facets/export/object/CDN; unauthorized branch, forged tenant header, current/revoked membership and stale grant; same Postgres pooled connection sequential A→B + rollback + background job; maintenance role/SECURITY DEFINER/views, stale placement epoch, tenant restore; offline-device permission revoked before sync; session/refresh theft, MFA and support break-glass. No test is claimed to have passed. Release gate pending MR-11/AR-013 and Security ADR approvals.

**Specific residual:** [MODULE_REGISTRY](../../MODULE_REGISTRY.md) assigns “Identity, membership, RBAC/ABAC authorization” to Security; [Security §17](../modules/SECURITY_ARCHITECTURE.md) and [ADR-MT-002](../adr/ADR-MT-002.md) prefer Multi-Tenant as writable membership owner; [ADR-BILL-010](../adr/ADR-BILL-010.md) wording can be read as Identity/Security membership source. Resolve single writer before schema/API work.

---

## 12. Database & Infrastructure Compatibility

Manbalar: [DB §7–11/§17](../modules/DATABASE_ARCHITECTURE.md), [Infra §3–12/§18](../modules/INFRA_ARCHITECTURE.md), [ADR-INFRA-002](../adr/ADR-INFRA-002.md), [ADR-DB-007](../adr/ADR-DB-007.md), [ADR-BE-005](../adr/ADR-BE-005.md).

- Stage A single-primary with encrypted offsite base backup + WAL/PITR + isolated restore; **not HA**. Stage B standby + monitored promotion + fencing. Stage C measured auto failover, optional read routing. SLO/RPO/RTO and initial stage not approved.
- Bounded pooled DB connections; runtime tenant context transaction-local if RLS selected; verify prepared statements and transaction pooling; primary for strongly consistent billing/stock/GL reads.
- Tenant-aware indexes, keyset paging, partition only after evidence, dedup uniqueness across partition boundaries; tenant placement registry/sharding optional and OPEN.
- Performance risk: hot SKU locks, high-volume immutable billing ledger, outbox/worker backlog, slow cross-domain reporting/search, offline device bursts, media uploads, heavy-tenant noisy neighbor, RLS overhead, WAL growth, primary contention, cache auth staleness.
- Quality gate: per-tenant peak/soak/concurrent sale+stock/billing month-end, DB locks/pool wait, p95/p99 and throughput, cost per workload; backup chain restore/drills and failover when selected. **No achieved latency/TPS/restore RPO/RTO benchmark is claimed**.

---

## 13. Sales, Inventory & Finance Lifecycle

**Current contract (PROPOSED; AR-005 / MR-07/08):** [Sales §11.2–11.3](../modules/SALES_INVENTORY.md), [Finance reconciliation](../modules/FINANCE_ARCHITECTURE.md), [Database §6/§17](../modules/DATABASE_ARCHITECTURE.md), [ADR-SI-012](../adr/ADR-SI-012.md), [FIN-002](../adr/FIN-002.md).

| Committed lifecycle fact | Physical inventory | ERP Finance/GL | SaaS Billing | Approval state |
|---|---|---|---|---|
| Sales draft / submit | No physical movement | No default journal | Charge eligibility OPEN | PROPOSED |
| Sales order confirm | Optional reservation; **no physical debit** | AR/GL not presumed | Document charge trigger OPEN | PROPOSED |
| Reservation / release / expiry | Available quantity changes; on-hand unchanged | No default posting | Classification OPEN | PROPOSED |
| Dispatch posted | Exactly one StockMovement/StockBalance effect | COGS/valuation/recognition trigger OPEN | Only independently approved source-document charge | PROPOSED |
| Invoice posted | **No second stock effect** | AR/revenue/tax only on approved policy | Independent Billing eligibility OPEN | PROPOSED |
| Goods receipt posted | Exactly one accepted stock increment | AP/GRNI and valuation trigger OPEN | Document type policy OPEN | PROPOSED |
| Return / correction | Compensating linked physical effect if original exists | Credit note/linked GL reversal only if appropriate | Separate credit/adjustment, **not automatic refund** | PROPOSED |

**Remaining text conflict (HIGH):** [Database §6](../modules/DATABASE_ARCHITECTURE.md) still groups SalesDocument posting, StockMovement and Finance journal within one DB transaction, while §17 and domain contracts explicitly separate typed domain facts and owner-specific ACID+outbox/retry boundaries. This can yield incompatible generated orchestration. **Next owner:** 03+04+05+06; select a canonical transaction matrix, leave obsolete proposal visibly historical without silently rewriting approved data.

**Concurrency/monetary invariant proposals:** append-only StockMovement, atomically updated StockBalance, reservation row locks/conditional updates, deterministic lock order, stable business idempotency keys, Finance journal debit/credit consistency and original-linked reversals. [ADR-SI-002](../adr/ADR-SI-002.md), [ADR-SI-003](../adr/ADR-SI-003.md), [ADR-DB-004](../adr/ADR-DB-004.md). **Untested**; valuation, negative stock, reservation TTL, journal recognition remain OPEN.

---

## 14. Mobile Offline & Integration Compatibility

**Mobile/Offline (AR-006/009, MR-12):** [ADR-MOB-004](../adr/ADR-MOB-004.md), [ADR-MOB-009](../adr/ADR-MOB-009.md), [MOBILE_ARCHITECTURE](../modules/MOBILE_ARCHITECTURE.md), [MOBILE_OFFLINE](../modules/MOBILE_OFFLINE.md) define scoped offline outbox, original operation_id, distinct accepted/rejected/conflict/retryable/unknown, reauthorization on reconnect and independent server acceptance. Revoked offline devices cannot be guaranteed instantly wiped; token TTL, quarantine/deletion and long-offline retention OPEN. No offline stock/GL/SaaS billing posting is authorized.

**Billing attribution from offline and external docs:** [Billing §19.2–19.4](../modules/BILLING_ARCHITECTURE.md), [ADR-BILL-003](../adr/ADR-BILL-003.md) separate untrusted device occurrence, server-accepted fact, durable receipt, rating and posting clocks. Calendar-month attribution, closed invoice adjustment and imported historical charge eligibility are **OPEN BUSINESS**; actual usage/negative one-calendar-month invariant and no credit ceiling remain intact.

**Integrations:** [INTEGRATION_ARCHITECTURE](../modules/INTEGRATION_ARCHITECTURE.md), [ADR-INT-002](../adr/ADR-INT-002.md), [ADR-INT-003](../adr/ADR-INT-003.md), [ADR-INT-004](../adr/ADR-INT-004.md) define tenant/provider/connection-scoped immutable external IDs, staged mapping, reconciliation and guarded outbound single-writer approach **PROPOSED**. No SalesDoc LIVE_VERIFIED API result or production cutover evidence. 

**AR-011 taxonomy harmonized:** [Integration Architecture](../modules/INTEGRATION_ARCHITECTURE.md) currently categorizes SalesDoc as **DOCUMENTED, not LIVE_VERIFIED**. [ADR-INT-001](../adr/ADR-INT-001.md) adopts same taxonomy; production auth, endpoint behavior and bidirectional write evidence not tested.

**Additional Round 2 watch, not a separate historical AR count:** Baseline [old Integration Architecture at baseline](https://github.com/yakhyodev/erp-saas-architecture/blob/016573c8426ed3ea3b1eacbd0dc81adb67861220/docs/modules/INTEGRATION_ARCHITECTURE.md) contained external provider documentation links, method/capability matrix, method-specific throttles, CS_id/SD_id/code_1C nuances, response errors, migration details. Current [Integration Architecture](../modules/INTEGRATION_ARCHITECTURE.md) is a much shorter reconciled taxonomy/contract memo and has **lost much of that source-specific evidence in its primary module doc**. Current [ADR-INT-001…005](../adr/) still summarize normalization, mappings, staging and cutover, but they do not replace that detailed provider capability catalog. **Recommendation:** preserve original documentation-derived evidence in a separate versioned PROPOSED provider appendix/compatibility matrix; explicitly distinguish DOCUMENTED from LIVE_VERIFIED. This regression is a **documentation traceability gap**, not proof of API failure. Do not silently assert method throttles are live-verified.

---

---

## 15. UX/UI Quality Review

Manbalar: [UX §1, §3–11, §16](../modules/UX_ARCHITECTURE.md), [ADR-UX-002](../adr/ADR-UX-002.md), [ADR-UX-003](../adr/ADR-UX-003.md), [ADR-UX-004](../adr/ADR-UX-004.md), [ADR-UX-006](../adr/ADR-UX-006.md), [Localization §15](../modules/LOCALIZATION_ARCHITECTURE.md).

**Current documented quality gate:**
- Role-aware Today/dashboard, “All Functions” discoverability, beginner/advanced views without permissions change, contextual primary action, Ctrl/Cmd+K search, power-user keyboard/bulk table.
- Single tenant/company/branch active context; stale in-flight queries and previous branch cache cannot repaint; deep-link resource permission checked server-side; All-branch only explicit grant.
- Global search for products, customers, orders, documents, payments, employees; authorized results **and counts, facets, recents, suggestions, history, snippets**, and indices/caches/reindex.
- Workflow-first drafted/approved/posted semantics for sale, receipt, return, transfer, AR/AP payments, offline visit/order/delivery, billing balance/top-up and permissions.
- Device-specific layout with shared domain meaning; desktop/mobile/offline states rejected/conflict/pending/unknown separated; no false offline “settled” feedback.
- Uzbek, Russian, English quality/terminology; hidden new-alphabet OFF; Unicode originals preserved; keyboard/screen reader/contrast/focus/zoom/reflow and WCAG 2.2 AA target.

**Top-20 candidate matrix now exists:** UX-T01…UX-T20 in [UX §16.1](../modules/UX_ARCHITECTURE.md). Its candidate personas, paths and forbidden transitions are **not a user-approved MVP list or verified usability study**. Proposed (unapproved) 2–3 navigation-action discoverability, p75 search <1s, LCP<=2.5s, INP<=200ms, CLS<=0.1 are targets, not measured success. UX release gates G0…G5 (§16.6) remain open. Need real role/device/network/dataset/locale testing and comparison before superiority over Sales Doctor can be claimed.

---

## 16. Localization & Alphabet Review

**Approved language scope:** uz/en/ru public interfaces, plus an internal hidden option for future Uzbek alphabet; no automatic public fourth locale. [DECISIONS BIZ-009](../../DECISIONS.md), [LOCALIZATION_ARCHITECTURE §1/§7/§15](../modules/LOCALIZATION_ARCHITECTURE.md).

**Current PROPOSED versioned contracts:** [ADR-LOC-001](../adr/ADR-LOC-001.md) BCP47-like public IDs and preference/fallback, [ADR-LOC-002](../adr/ADR-LOC-002.md) original business names/document/amount/ID preservation, [ADR-LOC-003](../adr/ADR-LOC-003.md) NFC and reversible search-only Uzbek apostrophe/Latin/Cyrillic aliases, [ADR-LOC-004](../adr/ADR-LOC-004.md) hidden default-OFF alphabet feature flag with internal preview.

- Locale switch **must not** alter canonical Product/SKU, INN/barcode, financial values, document numbers, tenant IDs, event identity or source evidence.
- Authorized global search includes results, counts, recents, suggestions, caches and reindex jobs; normalization aliases cannot leak tenant/branch data or silently mutate originals. [UX §16.3](../modules/UX_ARCHITECTURE.md), [Security §19](../modules/SECURITY_ARCHITECTURE.md).
- Mobile offline locale bundles/fallback and web UI must share stable error codes and translation keys, but do not make runtime locale authority for operations.
- **OPEN:** platform default, exact alias/transliteration collision policy, normalizer/index cutover, official future alphabet rules, device/locale/a11y benchmark, hidden rollout approval. No tests or public activation claimed.

**Result:** **HARMONIZED_PROPOSED**, no demonstrated locale or usability test success. **Owner:** 13+08+09+10+04. **Next:** agreed language QA scenarios and versioned index acceptance matrix.

---

## 17. Fixed Issues

1. **AR-006 file existence:** 2 Mobile ADR yaratilgan va ularda yetarli PROPOSED conflict/revocation asoslari bor. **Remaining:** approval va runtime tests.
2. **AR-007 file existence:** beshta registry yaratilgan. **Remaining:** hujjatlararo status sinxronlash, MR ro‘yxati, individual ADR approvals va oldingi “missing” matnlarini yangilash.
3. **AR-012 acceptance matrix:** bu faqat “missing matrix” documentary gapni yopadi, **butun AR-012ni yoki real UX quality gate’ni yopmaydi**.
4. **AR-003 va AR-011:** document taxonomy / staged proposals harmonized, **service architecture yoki external production verification complete degani emas**.

**Approval/evidence pending AR cases: 11/13 (AR-001/002/003/004/005/008/009/010/011/012/013).** AR-006 ham ayrim auth business/technical consentni kutmoqda; shu sabab 11 soni faqat “asosiy tarixiy topilmalar hozircha yopilmagan” klassifikatsiyasi, barcha qolgan individual qarorlar soni emas.

---

---

## 18. Remaining Open Questions

**Historical issue-level accounting:** 13 original AR findings remain in the issue register. Exactly **2 documentation existence gaps** are fixed (AR-006 two ADR files; AR-007 five root registries); **11 other AR findings** still require domain sign-off/user choice/evidence. Fixed file existence **does not close AR-006 policy or AR-007 registry synchronization**, so issue-level fully signed-off closures are **0/13**. AR-012’s 20-workflow **matrix itself** is now documented, but acceptance is not passed.

**All 20 MR requirements remain OPEN in some form**; they now have candidate technical contracts in multiple owning modules but lack an independently completed approval/acceptance ledger. Detailed status, owner and links are in **§7**. Do not present 20 as 20 equal business approvals — MR-10/11/15/17/19 include technical sign-off and test evidence, and MR-18 is MVP scope choice.

**Critical priority:** AR-001 payer/wallet/cardinality; AR-004 FIN-001 provenance; AR-005 posted lifecycle and GL policy; AR-008 Party/Payer/ReceivableOwner; AR-009 time attribution/negative month; AR-010 chargeable documents; AR-013 membership single owner; plus MR-02/03/05/06/11/14/16 approvals. [Central open questions](../../OPEN_QUESTIONS.md), [Billing §19](../modules/BILLING_ARCHITECTURE.md), [Finance addendum](../modules/FINANCE_ARCHITECTURE.md), [Security §17–19](../modules/SECURITY_ARCHITECTURE.md).

**Documentation carryover:** [OPEN_QUESTIONS](../../OPEN_QUESTIONS.md) still calls AR-006/007 missing, although real files now exist; its MR-01…MR-20 canonical status/owner register is incomplete. Corrections proposed under **§20**, not executed.

---

## 19. Risk Register & Severity

The severity below is the **current design-risk categorization** of the existing 13 AR issues, not independently confirmed production defects or new APPROVED conflicts. Each row’s evidence/owners/actions are documented in **§6**.

| Risk tier | AR IDs | Count | Predominant impact |
|---|---|---:|---|
| **HIGH** | AR-001, AR-002, AR-004, AR-005, AR-008, AR-009, AR-010, AR-013 | **8** | Money ownership/attribution, duplicate stock/GL/charge, identity and rights boundary |
| **MEDIUM** | AR-003, AR-006, AR-007, AR-011, AR-012 | **5** | HA stage, offline ADR/registry currency, external API evidence, UX acceptance |
| **CRITICAL confirmed** | — | **0** | No user-approved-vs-approved conflict or production breach proven in documentation audit |

**Additional Round-2 documentary watch items (NOT historical AR additions):**

| ID | Severity | Evidence | Owner / next action |
|---|---|---|---|
| R2-DOC-01 | MEDIUM | [OPEN_QUESTIONS](../../OPEN_QUESTIONS.md) AR-006/007 still “missing”; 97 ADRs not indexed with individual status and MR-01…20 missing as separate live tracking | 00+14: propose central-registry sync under separate authorization |
| R2-DOC-02 | MEDIUM | [Integration](../modules/INTEGRATION_ARCHITECTURE.md) shorter than [baseline provider evidence](https://github.com/yakhyodev/erp-saas-architecture/blob/016573c8426ed3ea3b1eacbd0dc81adb67861220/docs/modules/INTEGRATION_ARCHITECTURE.md) | 12: versioned DOCUMENTED provider evidence appendix, not false production verification |
| R2-TXN-01 | HIGH | [Database §6 vs §17](../modules/DATABASE_ARCHITECTURE.md) differences described in AR-005 | 03+04+05+06: canonical typed transaction/event/economic effect contract |

**Potential runtime threat classes only, not observed findings:** tenant data leakage through pool/search/cache/jobs, double-charge/reversal errors, hot inventory SKU contention, denied/revoked offline sync, lost WAL/PITR, HA failover split-brain, UX authorization side-channels, localization alias collisions. Test gates are PROPOSED and NOT EXECUTED. See **§10–§16**.

---

## 20. Required Decisions & Responsible Modules

The following are **recommendations only**. Every alternative below requires product/domain owner review, user approval for business consequences and an ADR status update with source evidence.

| Priority | Question for user | Recommended PROPOSED default / alternative | Why, risk, source |
|---|---|---|---|
| **P0-1** | Bir tenant nechta legal Company va SaaS payerni qamraydi? | **Candidate:** Tenant 1:N Company; Company 1:N Branch; **per-company BillingAccount 1:1 Wallet** for MVP, explicit payer assignment snapshot. Enterprise consolidated accounts deferred until approved. Alternative tenant-wide pooled wallet. | Simpler accounting isolation, but more wallets/seat attribution work. [Billing §19.1](../modules/BILLING_ARCHITECTURE.md), [MT §15](../modules/MULTI_TENANT_ARCHITECTURE.md). |
| **P0-2** | “Bir kalendar oy” minus davr qachon boshlanib tugaydi, qaysi timezone? | Compare **named calendar month** vs **next month same local date by calendar arithmetic** (not rolling 30 days); prefer explicit half-open UTC interval derived from approved business timezone. **Do not select either without user consent.** | A late-month first negative produces materially different grace; no forecast ceiling under any option. [Billing §19.4](../modules/BILLING_ARCHITECTURE.md). |
| **P0-3** | Qaysi hujjatlar va qaysi state uchun base fee? | **Candidate:** per-type stable **POSTED/FINALIZED** canonical occurrence rather than every draft, one base fee per canonical logical document; partial legs never auto-repeat base, correction via linked adjustment. Alternatives confirm/create. | Prevents accidental duplicate/abandoned draft charges; actual billable eligibility stays open. [ADR-BILL-009](../adr/ADR-BILL-009.md), [Sales §11.4](../modules/SALES_INVENTORY.md). |
| **P0-4** | Seat kimdan va qachondan hisoblanadi? | **Candidate:** activated membership, one charged seat per chosen billing account and approved time interval, effective-dated snapshots; prorating requires separate selection. | Invitation or branch transfer may otherwise double bill; [ADR-BILL-010](../adr/ADR-BILL-010.md). |
| **P0-5** | Finance revenue/COGS, tax, AR qachon post qilinadi? | **Candidate:** Invoice triggers AR/revenue under jurisdiction policy; physical dispatch triggers Inventory once and eventual authorized COGS/valuation; append-only corrections. Final tax/legal rules required. | Prevent double stock posting and premature revenue. [Finance addendum](../modules/FINANCE_ARCHITECTURE.md), [Sales §11.2](../modules/SALES_INVENTORY.md). |
| **P0-6** | Party/Customer/Payer/Outlet master kimga tegishli? | **Candidate:** one tenant-scoped canonical Party identity with CRM owning relationship/Outlet roles and Finance owning only debtor open items; external IDs mapped separately. Separate shared Party bounded context alternative. | Debtor/payer merges and cross-company payments are high risk. [CRM §11](../modules/CRM_DISTRIBUTION.md), [Finance](../modules/FINANCE_ACCOUNTING.md). |
| **P0-7** | Membership, roles, security policy yozuvlari kimniki? | **Candidate:** IdentityAuth→global principal/session; Multi-Tenant→TenantMembership; Security→RoleAssignment/Policy; Backend→enforce; no dual writer. | Align [Security §17](../modules/SECURITY_ARCHITECTURE.md) with [MODULE_REGISTRY](../../MODULE_REGISTRY.md) and [ADR-BILL-010](../adr/ADR-BILL-010.md). |
| **P1-8** | Catalog global, tenant-level yoki company overlay bo‘ladimi? | **Candidate:** tenant-scoped Catalog logical context + company-specific price/assortment mapping; separate Catalog writer, CRM relationship-only. | Low ops complexity; catalog sharing may be revisited for enterprise. [ADR-SI-011](../adr/ADR-SI-011.md). |
| **P1-9** | Kechikkan offline usage qaysi oyga kiradi? | **Candidate:** trusted server-effective business time, closed periodga kech kelsa current-period explicit adjustment referencing original; device time never authoritative. Alternative first-receipt month. | Maintains immutable invoices, no fake backdating; reconcile tier/negative grace. [Billing §19.3](../modules/BILLING_ARCHITECTURE.md). |
| **P1-10** | DB Stage A single primary yoki B standby bilan boshlanadimi? | **Candidate:** Stage A encrypted offsite WAL/PITR + tested restore if downtime contract accepts; Stage B if RPO/RTO/SLO requires HA. | Stage A HA emas, objectively measure tradeoff. [Infra §18](../modules/INFRA_ARCHITECTURE.md), [ADR-DB-007](../adr/ADR-DB-007.md). |
| **P1-11** | MVPning 20 ta workflow va UX gate raqamlari? | **Candidate:** [UX-T01…T20](../modules/UX_ARCHITECTURE.md#161-candidate-persona--top-20-workflow-acceptance-matrix-mr-16)ni pilot scope sifatida ko‘rib chiqish, role/device/locale dataset bilan usability test; 2–3 interaction maqsadini PROPOSED saqlash. | Better-than-Sales-Doctor claim needs task-specific comparable evidence. |
| **P1-12** | Go/PostgreSQL/Next.js/Flutter va modular monolith bo‘yicha ADR approval? | **Candidate:** Go+PostgreSQL+Next.js+Flutter va modular monolith after owner/toolchain/cost evidence review; tech **not APPROVED now**. | Strong current design rationale, but no explicit formal approval. [DECISIONS TECH](../../DECISIONS.md), [ADR-BE-001](../adr/ADR-BE-001.md). |
| **P1-13** | FIN-001ning asl approval manbasi bormi? | **Action:** Dated explicit user confirmation/link or formal ADR status review; keep original existing APPROVED label unchanged meanwhile. | Prevent fabricated provenance. [FIN-001](../adr/FIN-001.md), [DECISIONS](../../DECISIONS.md). |
| **P2-14** | Integration LIVE_VERIFIED requirements va outbound direction? | **Candidate:** documented-only adapter research; no write sync before owner-approved sandbox/live verification, method capability catalog, cutover evidence. | Avoid corrupt import/debt/stock. [Integration](../modules/INTEGRATION_ARCHITECTURE.md), [ADR-INT-004](../adr/ADR-INT-004.md). |
| **P2-15** | MVP accounting jurisdiction, fiscal currency, inventory negative/reservation policy, privacy retention? | **Action:** product/legal/accounting sign-off, separate scope matrix, do not default unapproved rules. | [Finance](../modules/FINANCE_ARCHITECTURE.md), [Sales §11.5](../modules/SALES_INVENTORY.md), [CRM](../modules/CRM_DISTRIBUTION.md). |

**Architecture-only technical sign-off (also blocking even after business choices):** reconcile Database §6 vs §17 atomicity; RLS/connection ownership/isolation test contracts; API/event version governance; finance idempotent GL; tenant move/backup recovery; privacy/security/UX acceptance protocols. These are technical ADR decisions, not automatically new user-approved business policy.

---

### Central documentation sync patches — proposed, NOT APPLIED

Only this new Round 2 review may be committed now; below are proposed documentation deltas for a later separately approved task:

| File | Precise recommended delta |
|---|---|
| [OPEN_QUESTIONS.md](../../OPEN_QUESTIONS.md) | AR-006 “missing ADRs” → DOCUMENTATION_FIXED but pending mobile policy; AR-007 “missing registries” → DOCUMENTATION_FIXED but pending synchronization; retain AR-001..013 as issue-history, add sub-status and closure evidence. Add MR-01..MR-20 with owner/business-vs-technical status. |
| [DECISIONS.md](../../DECISIONS.md) | Recount 97 ADR, cross-reference new BILL-009/010, SI-011/012/013, MOB-004/009 and all existing ADRs as PROPOSED unless otherwise justified; FIN-001 APPROVED (unverified provenance) unchanged pending user. |
| [REQUIREMENTS.md](../../REQUIREMENTS.md) | Bind MR-01…20 as OPEN/PROPOSED acceptance criteria; fix stale “missing” wording; no invented price or SLO. |
| [MODULE_REGISTRY.md](../../MODULE_REGISTRY.md) | Explicit IdentityCredentials vs TenantMembership vs RoleAssignment vs PermissionEvaluation, single writer candidate; Catalog/Pricing/Party/Payer domain map PROPOSED with dispute status. |
| [CHANGELOG.md](../../CHANGELOG.md) | Record corrections documentation events separately from approvals; mention Round 2 audit SHA and difference from historical baseline when user authorizes synchronization. |
| [docs/modules/MASTER_ARCHITECTURE.md](../modules/MASTER_ARCHITECTURE.md) | Link fresh Review Round 2, conditional ownership and stage-model references. Remain PROPOSED until approvals. |
| [docs/modules/DATABASE_ARCHITECTURE.md](../modules/DATABASE_ARCHITECTURE.md) | Clarify §6 “one transaction” vs §17 separate owned transaction+outbox without hiding historical alternative; no unilateral policy change. |
| [docs/modules/INTEGRATION_ARCHITECTURE.md](../modules/INTEGRATION_ARCHITECTURE.md) | Restore/version earlier documented SalesDoc source-specific method/capability/error/rate-limit evidence in a clearly DOCUMENTED appendix, never label LIVE_VERIFIED without tests. |

**No patches above executed by Round 2.**

---

---

## 21. Codex Implementation Readiness

| Gate | Evidence at current HEAD | Verdict | Required close condition |
|---|---|---|---|
| **GATE-01** Decision status/provenance | [DECISIONS](../../DECISIONS.md) created; [FIN-001](../adr/FIN-001.md) status without independent user proof; ADR coverage incomplete | **BLOCKED** | Source-linked approval record and complete ADR registry; FIN-001 provenance |
| **GATE-02** Ownership/cardinality | [MT §15](../modules/MULTI_TENANT_ARCHITECTURE.md), [Sales §11](../modules/SALES_INVENTORY.md), [CRM §11](../modules/CRM_DISTRIBUTION.md), [Security §17](../modules/SECURITY_ARCHITECTURE.md) | **BLOCKED** | Resolve BillingAccount, Catalog, Party, Membership canonical writers |
| **GATE-03** Tenant/security | [ADR-MT-001](../adr/ADR-MT-001.md), [Security §19](../modules/SECURITY_ARCHITECTURE.md) proposed RLS/guard tests | **BLOCKED** | Isolation ADR decision; auth scope sign-off; test contract accepted |
| **GATE-04** Billing integrity | [Billing §19](../modules/BILLING_ARCHITECTURE.md), [ADR-BILL-009](../adr/ADR-BILL-009.md), [ADR-BILL-010](../adr/ADR-BILL-010.md) | **BLOCKED** | Approve seat, document, discount, wallet, negative month/late usage |
| **GATE-05** Sales/Stock/Finance | [Sales §11.2](../modules/SALES_INVENTORY.md), [FIN-002](../adr/FIN-002.md), [DB §6/§17](../modules/DATABASE_ARCHITECTURE.md) | **BLOCKED** | One transaction/event model and Finance GL/AR/COGS/tax trigger |
| **GATE-06** API/offline/integrations | [Backend §17](../modules/BACKEND_ARCHITECTURE.md), [MOB-004/009](../modules/MOBILE_ARCHITECTURE.md), [Integration](../modules/INTEGRATION_ARCHITECTURE.md) | **BLOCKED** | Stable signed-off API/event/adapter versions, external evidence, conflict policy |
| **GATE-07** Infra/DR/perf | [Infra §18](../modules/INFRA_ARCHITECTURE.md), [ADR-DB-007](../adr/ADR-DB-007.md) | **BLOCKED** | Production Stage A/B, SLO/RPO/RTO and agreed load/restore test plan |
| **GATE-08** UX/UI | [UX §16](../modules/UX_ARCHITECTURE.md), [Localization §15](../modules/LOCALIZATION_ARCHITECTURE.md) | **READY_WITH_CONDITIONS as design inputs; gate BLOCKED** | MVP task/role approval and measurable UX/a11y/test profiles |
| **GATE-09** Documentation consistency | Five registries exist; [OPEN_QUESTIONS](../../OPEN_QUESTIONS.md) stale AR-006/007, MR tracking missing; root detailed ADR index incomplete | **BLOCKED** | Reconcile statuses, MR-01…20, cross-module links and sign-offs |

**Gate status is an architecture-only prerequisite**, not a claim of production tests already passing. Approved implementation readiness differs from operational/production readiness.

---

### Module-by-module readiness

| # | Modul | Result | Dalil / blocking condition |
|---|---|---|---|
| 00 | Master Architecture | **BLOCKED** | [Master §15](../modules/MASTER_ARCHITECTURE.md), [registries](../../DECISIONS.md): ownership, registry, provenance |
| 01 | Billing | **BLOCKED** | [Billing §19](../modules/BILLING_ARCHITECTURE.md): rating/Seat/month/account choices |
| 02 | Multi-Tenant | **BLOCKED** | [MT §15](../modules/MULTI_TENANT_ARCHITECTURE.md): cardinality and membership/placement |
| 03 | Database | **BLOCKED** | [DB §6/§17](../modules/DATABASE_ARCHITECTURE.md): transaction contradiction, HA/isolation choice |
| 04 | Backend | **BLOCKED** | [Backend §17](../modules/BACKEND_ARCHITECTURE.md): canonical contracts, approved platform stack |
| 05 | Sales & Inventory | **BLOCKED** | [Sales §11](../modules/SALES_INVENTORY.md): ownership, GL trigger, stock/charge policy |
| 06 | Finance & Accounting | **BLOCKED** | [Finance addendum](../modules/FINANCE_ARCHITECTURE.md), [FIN-001](../adr/FIN-001.md): tax, GL, provenance |
| 07 | CRM & Distribution | **BLOCKED** | [CRM §11](../modules/CRM_DISTRIBUTION.md): Party/payer scope and data governance |
| 08 | UX/UI | **READY_WITH_CONDITIONS** | [UX §16](../modules/UX_ARCHITECTURE.md): concrete matrix, unapproved role scope and unmeasured user tests |
| 09 | Mobile & Offline | **BLOCKED** | [MOB-004](../adr/ADR-MOB-004.md)/[009](../adr/ADR-MOB-009.md): present, but offline TTL/authorization policy unapproved |
| 10 | Security & Permissions | **BLOCKED** | [Security §17–19](../modules/SECURITY_ARCHITECTURE.md): membership authority, RLS, auth choices |
| 11 | Infrastructure & Scaling | **BLOCKED** | [Infra §18](../modules/INFRA_ARCHITECTURE.md): Stage A/B, numeric SLO/RPO/RTO |
| 12 | Integrations | **BLOCKED** | [Integration](../modules/INTEGRATION_ARCHITECTURE.md): live evidence, import/cutover policy, provider appendix |
| 13 | Localization & Alphabet | **READY_WITH_CONDITIONS** | [Localization §15](../modules/LOCALIZATION_ARCHITECTURE.md): language readiness + proposed search/locale/hidden rollout and QA |

**Sum: 12 BLOCKED, 2 READY_WITH_CONDITIONS, 0 READY. Whole ERP SaaS: BLOCKED for Codex product-code implementation.** Individual design-review tasks may continue, not executable application development without separate user authorization.

---

**Codex tools:** skills and MCP are available for creative research/implementation *after* formal authorization; they cannot independently turn an OPEN business policy or technical ADR into APPROVED. **Ready** means ready to enter architecture-authorized implementation, not production proven.

---

## 22. Final Audit Verdict

**High residual risk:** economic correctness (wallet/payer/negative month/pricing), stock/Finance GL event-time reconciliation, canonical Party/permissions ownership, mutable source migration, tenant isolation under pool/worker, UX state mismatch. **Medium residual risk:** HA scope without numeric target, performance hot paths, mobile device long-offline privacy, catalog/salesdoc capability loss, registry governance drift. **No production vulnerability or user-approved rule conflict was independently proven** by this document-only review.

**Yakuniy verdict: `BLOCKED` for Codex product implementation.** Architecture as a design is substantially better documented than at baseline. Specifically **2 historical missing-document cases are physically fixed (AR-006, AR-007)**, typed workflows and UX matrix documented, and many competing proposals harmonized; **no new business/technical approval** derived from those commits. Eleven remaining historical AR matters require formal approval/sign-off or independent evidence; AR-006 and AR-007 still need registry status cleanup and AR-006 also auth policy approvals.

**Recommended work order:** (1) user P0 decision packet; (2) domain owners unify actual ownership + lifecycle + billing month/charge policy; (3) technical ADR alignment on database transaction, auth/RLS, HA and API; (4) master/central registry status/provenance patch by separate user authorization; (5) UX top-20 usability and security/performance test plans with real baselines; (6) Round 3 audit; (7) **only explicit implementation permission** allows Codex to write product code.

**Governance:** Do not change historical FIN-001 label without evidence/user action. Do not convert PROPOSED technical options into approved invariants. Do not assume measured benchmarks or tests passed. Do not claim Sales Doctor superiority before same-task comparison. No modifications to branch, repo description, modules, existing ADRs, root registries or historical review are authorized by this Round 2.

**Final Round 2 disposition:** `REVIEWED — CORRECTIONS REQUIRED` / Codex `BLOCKED`. Fixed documentation gaps: **2**. Original issue register: **13**, all still have at least policy/provenance/registry follow-up, with 11 not documentary closed. MR open-register: **20**, requiring appropriately scoped business or technical decisions. The old report remains historically preserved; only this report was editorially reorganized.

---

## 23. References & Evidence

### Immutable commit evidence

- [Audit baseline 016573c8426ed3ea3b1eacbd0dc81adb67861220](https://github.com/yakhyodev/erp-saas-architecture/tree/016573c8426ed3ea3b1eacbd0dc81adb67861220) — 108 Markdown, 90 ADR at baseline; no claim this is current main.
- [Audited Round-2 source 184b32f7b3dcc9ce8d329ca2dfb7895f06cae905](https://github.com/yakhyodev/erp-saas-architecture/tree/184b32f7b3dcc9ce8d329ca2dfb7895f06cae905) — 121 Markdown, 97 ADR, 17 modules and five registries.
- [Previous Round-2 review commit 776d5190b696c776ebaa2d0a0549daac9b32873b](https://github.com/yakhyodev/erp-saas-architecture/commit/776d5190b696c776ebaa2d0a0549daac9b32873b) — previous complete report version retained in GitHub history.
- [Historical first audit](ARCHITECTURE_REVIEW.md) — original AR/MR gap definitions, not current sign-off.

### Central governance

[DECISIONS](../../DECISIONS.md) · [REQUIREMENTS](../../REQUIREMENTS.md) · [MODULE_REGISTRY](../../MODULE_REGISTRY.md) · [OPEN_QUESTIONS](../../OPEN_QUESTIONS.md) · [CHANGELOG](../../CHANGELOG.md) · [README](../../README.md).

### Modules and evidence crosswalk

| Direction | Primary current source | Relevant ADR examples |
|---|---|---|
| 00 Master | [Master Architecture](../modules/MASTER_ARCHITECTURE.md) | [ADR-002](../adr/ADR-002-domain-boundaries.md), [ADR-003](../adr/ADR-003-modular-monolith.md) |
| 01 Billing | [Billing §19](../modules/BILLING_ARCHITECTURE.md) | [BILL-001](../adr/ADR-BILL-001.md), [BILL-006](../adr/ADR-BILL-006.md), [BILL-009](../adr/ADR-BILL-009.md), [BILL-010](../adr/ADR-BILL-010.md) |
| 02 Multi-Tenant | [Multi-Tenant §15](../modules/MULTI_TENANT_ARCHITECTURE.md) | [MT-001](../adr/ADR-MT-001.md), [MT-002](../adr/ADR-MT-002.md) |
| 03 Database | [Database §6/§17](../modules/DATABASE_ARCHITECTURE.md) | [DB-003](../adr/ADR-DB-003.md), [DB-004](../adr/ADR-DB-004.md), [DB-007](../adr/ADR-DB-007.md) |
| 04 Backend | [Backend §17](../modules/BACKEND_ARCHITECTURE.md) | [BE-003](../adr/ADR-BE-003.md), [BE-006](../adr/ADR-BE-006.md) |
| 05 Sales/Inventory | [Sales §11](../modules/SALES_INVENTORY.md) | [SI-011](../adr/ADR-SI-011.md), [SI-012](../adr/ADR-SI-012.md), [SI-013](../adr/ADR-SI-013.md) |
| 06 Finance | [Finance architecture](../modules/FINANCE_ARCHITECTURE.md), [Finance accounting](../modules/FINANCE_ACCOUNTING.md) | [FIN-001](../adr/FIN-001.md) (provenance unverified), [FIN-002](../adr/FIN-002.md) |
| 07 CRM | [CRM §11](../modules/CRM_DISTRIBUTION.md) | [CRM-001](../adr/ADR-CRM-001-domain-ownership.md), [CRM-003](../adr/ADR-CRM-003-salesdoc-adapter.md) |
| 08 UX | [UX §16](../modules/UX_ARCHITECTURE.md), [historical UX](../modules/UX_UI_ARCHITECTURE.md) | [UX-003](../adr/ADR-UX-003.md), [UX-004](../adr/ADR-UX-004.md), [UX-006](../adr/ADR-UX-006.md) |
| 09 Mobile | [Mobile](../modules/MOBILE_ARCHITECTURE.md), [Mobile offline](../modules/MOBILE_OFFLINE.md) | [MOB-004](../adr/ADR-MOB-004.md), [MOB-009](../adr/ADR-MOB-009.md) |
| 10 Security | [Security §17–19](../modules/SECURITY_ARCHITECTURE.md) | [SEC-002](../adr/ADR-SEC-002.md), [SEC-004](../adr/ADR-SEC-004.md) |
| 11 Infra | [Infra §18](../modules/INFRA_ARCHITECTURE.md) | [INFRA-002](../adr/ADR-INFRA-002.md), [INFRA-004](../adr/ADR-INFRA-004.md) |
| 12 Integrations | [Integration](../modules/INTEGRATION_ARCHITECTURE.md) | [INT-001](../adr/ADR-INT-001.md), [INT-005](../adr/ADR-INT-005.md) |
| 13 Localization | [Localization §15](../modules/LOCALIZATION_ARCHITECTURE.md) | [LOC-001](../adr/ADR-LOC-001.md), [LOC-003](../adr/ADR-LOC-003.md), [LOC-004](../adr/ADR-LOC-004.md) |

**Source-use limitation:** references establish what the GitHub documents propose; they do not establish real-world financial correctness, API LIVE_VERIFIED, a completed RLS/security test or quantitative UX superiority.
