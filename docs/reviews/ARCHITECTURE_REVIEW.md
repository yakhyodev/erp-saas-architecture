# ERP SaaS — FINAL CROSS-MODULE ARCHITECTURE REVIEW

> **Status:** REVIEWED — CORRECTIONS REQUIRED  
> **Loyiha:** Sales doc copy — ERP SaaS, 14 — Architecture Review  
> **Repository:** [yakhyodev/erp-saas-architecture](https://github.com/yakhyodev/erp-saas-architecture)  
> **Audit qilingan branch:** main  
> **AUDIT BASELINE COMMIT:** [016573c8426ed3ea3b1eacbd0dc81adb67861220](https://github.com/yakhyodev/erp-saas-architecture/commit/016573c8426ed3ea3b1eacbd0dc81adb67861220)  
> **Audit sanasi:** 2026-10-12 (hisobot yakunlangan vaqt)  
> **Yozishdan oldingi main HEAD:** 016573c8426ed3ea3b1eacbd0dc81adb67861220 — audit baseline bilan ayni commit.  
> **Joriy main HEAD va audit baseline farqi:** ushbu yangi review fayli alohida yangi commitda yaratiladi. Shuning uchun fayl yaratilgach joriy main HEAD yangi review commitga siljiydi, yuqoridagi 016573c8… esa **o‘zgarmas audit manbasi** bo‘lib qoladi. Review faylining o‘zida bo‘lajak commit SHA oldindan taxmin qilinmaydi; yakuniy commit yozish amaliyoti natijasida alohida tekshiriladi.  
> **Review doirasi:** GitHub’dagi hujjat/ADR matnlari; ishlab turgan dastur, database, infratuzilma va penetration test auditi emas.  
> **Implementatsiya hukmi:** BLOCKED; hozircha kod yozish uchun umumiy ruxsat yo‘q.

## Executive summary

Audit baseline asosida 108 ta Markdown fayl aniqlandi va tekshirildi: **14 ta arxitektura yo‘nalishi, 17 ta modul hujjati, 90 ta ADR va README.md**. ADR statuslari: **89 PROPOSED va 1 hujjat ichida APPROVED deb belgilangan** (FIN-001; alohida foydalanuvchi tasdig‘i repositoryda mustaqil isbotlanmagan). Besh asosiy umumiy hujjat yo‘q; Master hujjati faqat docs/modules katalogida PROPOSED modul hujjati sifatida mavjud. Mobile registrida ko‘rsatilgan ikkita ADR fayli mavjud emas.

**13 audit topilmasi: 8 HIGH, 5 MEDIUM, 0 tasdiqlangan CRITICAL.** Bu raqamlar quyidagi AR-001…AR-013 reyestriga mos. **20 ta ochiq/yetishmayotgan talab:** MR-01…MR-20. **Codex readiness:** 12 BLOCKED, 2 READY WITH CONDITIONS, 0 READY. Butun tizim uchun implementatsiya gate — BLOCKED.

**Muhim farqlash:** boshqa-boshqa PROPOSED texnik arxitektura variantlarining farqi, tasdiqlangan qarorlar o‘zaro ziddiyati degani emas. APPROVED biznes qoidalarini buzuvchi tasdiqlangan qaror isbotlanmadi. HIGH severity ayrim ochiq mavzular noto‘g‘ri implementatsiya qilinsa yuqori zarar keltirish xavfini anglatadi, productiondagi real xatolikni emas.

### Audit mezonlari va dalil chegarasi

1. Dalil — GitHub main branchidagi aniq modul fayli yoki alohida ADR. Tegishli manbalar ushbu hujjatda nisbiy havola bilan berilgan; bu havolalar GitHubdagi docs/reviews faylidan ishlaydi.
2. BIZ-001…BIZ-009 va foydalanuvchi tasdiqlagan UX ustuvorligi o‘zgarmas boshlang‘ich shart; bu qoidalar bu audit orqali qayta tasdiqlanmaydi yoki o‘zgartirilmaydi.
3. ADR ichidagi APPROVED yorlig‘i avtomatik tarzda foydalanuvchi tomonidan tasdiqlanganini isbotlamaydi.
4. Kod, migration, test runner, live external API, server konfiguratsiyasi, RLS policies, haqiqiy benchmark va production loglari sinovdan o‘tkazilmadi. Security/performance xulosalari design-readiness bahosidir.
5. Conflict turlari: **CONFIRMED PROPOSAL MISMATCH** (hujjatlarda turlicha taklif), **CONTRACT AMBIGUITY** (muhim bo‘shliq), **DOCUMENTATION/PROVENANCE GAP** va **UNRESOLVED REQUIREMENT**.
6. Muammolar tavsiya bilan beriladi. PROPOSED matnni APPROVED holatiga o‘tkazish yoki eski qarorni SUPERSEDED qilish faqat alohida foydalanuvchi tasdig‘i bilan mumkin.

---

# A. Documentation Inventory

## A1. Fayllar va haqiqiy tuzilma

GitHub main dagi rekursiv tree va alohida fayl mazmunlari asosida:

| Katalog/tur | Soni | Holat |
|---|---:|---|
| docs/modules/ | 17 Markdown | 14 arxitektura yo‘nalishini qamrab oladi; texnik takliflar PROPOSED |
| docs/adr/ | 90 Markdown | 89 PROPOSED, FIN-001 esa APPROVED deb yozilgan |
| README.md | 1 | Arxitektura planning haqida qisqa ta'rif |
| **Jami** | **108** | Audit baseline’dagi barcha Markdown fayllar |

[Repository tree](https://github.com/yakhyodev/erp-saas-architecture/tree/016573c8426ed3ea3b1eacbd0dc81adb67861220) — audit baseline. [README](../../README.md).

**MASTER_ARCHITECTURE.md** repository ildizida mavjud emas; real fayl [docs/modules/MASTER_ARCHITECTURE.md](../modules/MASTER_ARCHITECTURE.md). Uning kirish qismida bu modul hujjati root master o‘rnini bosmasligi va boshqa chatlar ADRlarini avtomatik APPROVED qilmasligi ko‘rsatilgan.

**main’da topilmagan besh asosiy hujjat:**

- REQUIREMENTS.md
- DECISIONS.md
- MODULE_REGISTRY.md
- OPEN_QUESTIONS.md
- CHANGELOG.md

Ularni biror boshqa branch yoki lokal faylda mavjud bo‘lishi ehtimoli bu baseline’da mavjud deb hisoblashga asos emas.

## A2. 14 yo‘nalish va 17 modul hujjati

| Chat | Hujjat(lar) | Hujjatdagi maqom / qamrov |
|---|---|---|
| 00 Master | [MASTER_ARCHITECTURE.md](../modules/MASTER_ARCHITECTURE.md) | PROPOSED, domain xaritasi va umumiy prinsiplari |
| 01 Billing | [BILLING_ARCHITECTURE.md](../modules/BILLING_ARCHITECTURE.md) | APPROVED biznes qoidalaridan alohida PROPOSED wallet, usage, ledger |
| 02 Multi-Tenant | [MULTI_TENANT_ARCHITECTURE.md](../modules/MULTI_TENANT_ARCHITECTURE.md) | PROPOSED tenant, company, branch, RLS, placement |
| 03 Database | [DATABASE_ARCHITECTURE.md](../modules/DATABASE_ARCHITECTURE.md) | PROPOSED relational ERD, isolation, transactions, backup |
| 04 Backend | [BACKEND_ARCHITECTURE.md](../modules/BACKEND_ARCHITECTURE.md) | PROPOSED modular monolith, API, worker, events |
| 05 Sales & Inventory | [SALES_INVENTORY.md](../modules/SALES_INVENTORY.md) | PROPOSED order, purchase, stock, reservation, workflow |
| 06 Finance | [FINANCE_ARCHITECTURE.md](../modules/FINANCE_ARCHITECTURE.md), [FINANCE_ACCOUNTING.md](../modules/FINANCE_ACCOUNTING.md) | Ikkita PROPOSED hujjat; ledgerlar ajralishi bo‘yicha FIN-001 tasdiq-provenance ochiq |
| 07 CRM & Distribution | [CRM_DISTRIBUTION.md](../modules/CRM_DISTRIBUTION.md) | PROPOSED party, agent, outlet, territory, route, KPI |
| 08 UX/UI | [UX_ARCHITECTURE.md](../modules/UX_ARCHITECTURE.md), [UX_UI_ARCHITECTURE.md](../modules/UX_UI_ARCHITECTURE.md) | PROPOSED; ikkinchisi historical concept |
| 09 Mobile & Offline | [MOBILE_ARCHITECTURE.md](../modules/MOBILE_ARCHITECTURE.md), [MOBILE_OFFLINE.md](../modules/MOBILE_OFFLINE.md) | PROPOSED; ikkinchisi companion |
| 10 Security | [SECURITY_ARCHITECTURE.md](../modules/SECURITY_ARCHITECTURE.md) | PROPOSED identity/security controls |
| 11 Infrastructure | [INFRA_ARCHITECTURE.md](../modules/INFRA_ARCHITECTURE.md) | PROPOSED deployment, HA, backup, DR |
| 12 Integrations | [INTEGRATION_ARCHITECTURE.md](../modules/INTEGRATION_ARCHITECTURE.md) | PROPOSED SalesDoc adapter, identity mapping, staging |
| 13 Localization | [LOCALIZATION_ARCHITECTURE.md](../modules/LOCALIZATION_ARCHITECTURE.md) | APPROVED tillar talabi + PROPOSED locale/search/rollout |

Takror mavzudagi hujjatlar alohida ko‘rildi: Finance ikkita hujjatida ledger ajratish va double-entry yo‘nalishi konseptual mos, lekin bu ikkita nomdan qaysi biri canonical ekanligi rasmiy reyestrda yo‘q; Mobile companion alohida qarorlarni almashtirmasligini aytadi; UX historical hujjati eski ADR-UX-001…003 yorliqlari canonical emasligini aniq yozadi.

## A3. ADR katalogi va tekshiruv qamrovi

| ADR oilasi | Soni | Maqom |
|---|---:|---|
| ADR-001…009, Master | 9 | PROPOSED |
| ADR-BILL-001…008 | 8 | PROPOSED |
| ADR-MT-001…002 | 2 | PROPOSED |
| ADR-DB-001…009 | 9 | PROPOSED |
| ADR-BE-001…008 | 8 | PROPOSED |
| ADR-SI-001…010 | 10 | PROPOSED |
| FIN-001…002 | 2 | FIN-001 hujjatda APPROVED, FIN-002 PROPOSED |
| ADR-CRM-001…005 | 5 | PROPOSED |
| ADR-UX-001…006 | 6 | PROPOSED |
| ADR-MOB-001…009 raqamlar qatori | 7 mavjud | 004 va 009 yetishmaydi, qolganlari PROPOSED |
| ADR-SEC-001…010 | 10 | PROPOSED |
| ADR-INFRA-001…005 | 5 | PROPOSED |
| ADR-INT-001…005 | 5 | PROPOSED |
| ADR-LOC-001…004 | 4 | PROPOSED |
| **Jami mavjud** | **90** | **89 PROPOSED, 1 APPROVED deb yozilgan** |

Mobile arxitektura [ADR registry](../modules/MOBILE_ARCHITECTURE.md#adr-registry) bo‘limida [ADR-MOB-004](https://github.com/yakhyodev/erp-saas-architecture/blob/main/docs/adr/ADR-MOB-004.md) (conflict resolution/idempotency) va [ADR-MOB-009](https://github.com/yakhyodev/erp-saas-architecture/blob/main/docs/adr/ADR-MOB-009.md) (auth/device security) keltirilgan, ammo baseline tree’da shu ikki fayl yo‘q. Bu manzillar mavjud faylga havola sifatida emas, **broken-reference dalili** sifatida ko‘rsatilmoqda.

### A4. ADR sifati

- **Aniqlik:** aksariyat ADRda muammo, variant, tanlov, sabab, xavf, ta’sir maydonlari bor; qisqa yozilgan ADRlar implementatsiya uchun to‘liq shartnoma emas.
- **Status:** deyarli hammasida PROPOSED aniq; FIN-001 ichida APPROVED yozuvi bor. Boshqa tasdiq faktini bu audit uydirmaydi.
- **Muqobillar:** asosan mavjud, ammo ayrim qisqa Security va boshqa ADRlarda variantlar faqat nomi bilan aytilgan.
- **Risk:** tilga olingan, lekin ko‘pincha o‘lchanadigan mitigation/test/SLO berilmagan.
- **Cross-module ta’sir:** keng qayd etilgan, ammo umumiy qarorlar reyestri va imzolangan kontrakt yo‘q.
- **Provenance:** FIN-001 uchun mustaqil foydalanuvchi tasdig‘iga konkret GitHub dalili yo‘q; statusni avtomatik o‘zgartirmaslik kerak.
- **History:** UX_UI_ARCHITECTURE historical ADR-UX-001…003 nomlari bilan canonical [ADR-UX-001…006](../modules/UX_ARCHITECTURE.md#12-architecture-decision-records) farqlangan; import/automatic extraction tarixiy matnni canonical deb o‘qimasligi kerak.

Manbalar: [FIN-001](../adr/FIN-001.md), [FIN-002](../adr/FIN-002.md), [UX_UI_ARCHITECTURE.md](../modules/UX_UI_ARCHITECTURE.md), [MOBILE_ARCHITECTURE.md](../modules/MOBILE_ARCHITECTURE.md), [ADR-MT-001](../adr/ADR-MT-001.md), [ADR-BILL-006](../adr/ADR-BILL-006.md).

---

# B. Architecture Consistency Matrix

**“Mos”** faqat PROPOSED modellar mantiqan bir-birini qo‘llashini anglatadi; texnik tanlov APPROVED yoki ishchi implementatsiya tekshirildi degani emas.

| Bog‘liqlik | Baholash | Dalil va izoh |
|---|---|---|
| Master ↔ Backend | Konseptual mos | Bounded context, modular monolith, gradual extraction: [Master §4–5](../modules/MASTER_ARCHITECTURE.md), [Backend §3–4](../modules/BACKEND_ARCHITECTURE.md) |
| Master ↔ Barcha modullar | Qisman | Global master, module registry va final domain ownership hali yo‘q: [Master §12–13](../modules/MASTER_ARCHITECTURE.md) |
| Multi-Tenant ↔ Database ↔ Security | Konseptual mos | Composite FK, server authorization, RLS defense-in-depth: [ADR-MT-001](../adr/ADR-MT-001.md), [ADR-DB-003](../adr/ADR-DB-003.md), [ADR-SEC-004](../adr/ADR-SEC-004.md) |
| Multi-Tenant ↔ Billing | PROPOSED ownership nomuvofiqligi | Wallet kompaniya bilan, billing account va tenant cardinality esa ochiq: [ADR-BILL-001](../adr/ADR-BILL-001.md), [Multi-Tenant §10](../modules/MULTI_TENANT_ARCHITECTURE.md) |
| Billing ↔ Finance | Qisman mos | SaaS va ERP ledgeri ajratish konsepsiyasi mos, FIN-001 approval provenance va settlement contract ochiq: [FIN-001](../adr/FIN-001.md), [Billing §2–4](../modules/BILLING_ARCHITECTURE.md) |
| Backend ↔ Database | Konseptual mos | ACID/outbox/idempotent consumer, bounded pool: [ADR-BE-003](../adr/ADR-BE-003.md), [ADR-DB-004](../adr/ADR-DB-004.md) |
| Sales ↔ Inventory ↔ Database | Qisman mos | Concurrency va ledger mos; order confirmation, dispatch va GL posting triggeri umumiy kontraktda aniqlanmagan: [Sales §1, §4, §10](../modules/SALES_INVENTORY.md), [Database §6](../modules/DATABASE_ARCHITECTURE.md) |
| Sales ↔ Finance ↔ CRM | Ownership noaniq | Counterparty, Catalog, payer/sale-point, AR masalalari: [ADR-CRM-001](../adr/ADR-CRM-001-domain-ownership.md), [ADR-INT-005](../adr/ADR-INT-005.md) |
| Backend ↔ Mobile ↔ Integrations | Qisman mos | Retry va idempotency mos, offline conflict/late billing va external live verification ochiq: [Mobile](../modules/MOBILE_ARCHITECTURE.md), [Integration](../modules/INTEGRATION_ARCHITECTURE.md) |
| Infrastructure ↔ Database | PROPOSED HA variantlari farqli | Boshlanish primary+standby yoki single primary+PITR: [Database §9](../modules/DATABASE_ARCHITECTURE.md), [Infra §7](../modules/INFRA_ARCHITECTURE.md) |
| UX ↔ Security ↔ Backend | Konseptual mos | UI-only authorization rad, server authoritative: [ADR-UX-002](../adr/ADR-UX-002.md), [ADR-SEC-002](../adr/ADR-SEC-002.md) |
| UX ↔ Mobile ↔ Localization | Konseptual mos | Workflow parity, offline statuslar, uz/en/ru va hidden locale: [UX §8–9](../modules/UX_ARCHITECTURE.md), [Localization §7](../modules/LOCALIZATION_ARCHITECTURE.md) |
| UX ↔ Biznes modullar | Qisman | Task-first namunalari bor, lekin tasdiqlangan top-20 task/workflow registry va sinov natijalari yo‘q: [UX §1, §6, §13](../modules/UX_ARCHITECTURE.md) |

### B1. Asosiy yaxshi mosliklar

**Tenant isolation:** Multi-Tenant → Database → Security uchligi app authorization, trusted tenant context, composite tenant key/FK, RLS va worker/cache/storage tekshiruvlarini bir-biriga zid bo‘lmagan qatlamlar sifatida ko‘radi. Branch permission RLS bilan avtomatik hal bo‘lmaydi. [ADR-MT-001](../adr/ADR-MT-001.md), [ADR-DB-003](../adr/ADR-DB-003.md), [ADR-SEC-003](../adr/ADR-SEC-003.md), [ADR-SEC-004](../adr/ADR-SEC-004.md).

**Billing correctness:** append-only ledger, immutable pricing version, at-least-once delivery + idempotent economic effect, verified settlement bilan wallet credit. [ADR-BILL-002](../adr/ADR-BILL-002.md), [ADR-BILL-003](../adr/ADR-BILL-003.md), [ADR-BILL-004](../adr/ADR-BILL-004.md), [ADR-BILL-005](../adr/ADR-BILL-005.md), [ADR-BE-003](../adr/ADR-BE-003.md).

**Inventory consistency:** immutable StockMovement, transactionally maintained StockBalance, concurrency-safe reservation, reversal; Sales/Inventory va DB konsepsiyalari uyg‘un. [ADR-SI-002](../adr/ADR-SI-002.md), [ADR-SI-003](../adr/ADR-SI-003.md), [ADR-SI-004](../adr/ADR-SI-004.md).

**Scaling:** darhol microservices/sharding/Kubernetes/Redis/Kafka majburiy emasligi va o‘lchovga asoslangan qarorlar [ADR-003](../adr/ADR-003-modular-monolith.md), [ADR-BE-008](../adr/ADR-BE-008.md), [ADR-INFRA-003](../adr/ADR-INFRA-003.md)da izchil taklif qilingan.

---

# C. Confirmed Decisions Audit

Quyidagi “Mos” belgisi GitHubdagi hujjat **foydalanuvchi oldin belgilagan** talabga mos yozilganini bildiradi; texnik yechim tasdiqlanganini emas.

| Talab | Manba | Natija |
|---|---|---|
| BIZ-001 — Hybrid Prepaid SaaS | [Billing §1](../modules/BILLING_ARCHITECTURE.md) | Mos |
| BIZ-002 — arzon Seat + Business Usage + Resource Usage | [Billing §1, §6–8](../modules/BILLING_ARCHITECTURE.md) | Mos |
| BIZ-003 — hujjat asosiy narxi + hajmdan keyin qo‘shimcha kredit | [Billing §7](../modules/BILLING_ARCHITECTURE.md) | Mos; trigger/units ochiq |
| BIZ-004 — 1 kredit = 10 so‘m | [Billing §1](../modules/BILLING_ARCHITECTURE.md), [Database §2](../modules/DATABASE_ARCHITECTURE.md) | Mos |
| BIZ-005 — progressiv hajm chegirmasi va enterprise shartnoma | [Billing §9–10](../modules/BILLING_ARCHITECTURE.md) | Mos; tier algorithm PROPOSED |
| BIZ-006 — balans tugaganda haqiqiy Seat va Usage bir kalendar oy minusga | [Billing §13](../modules/BILLING_ARCHITECTURE.md) | Mos; calendar lifecycle ochiq |
| BIZ-007 — minus balans uchun taxminiy kredit limiti yo‘q | [ADR-BILL-006](../adr/ADR-BILL-006.md) | Mos |
| BIZ-008 — kompaniya va filiallar yagona SaaSda | [Multi-Tenant §1–3](../modules/MULTI_TENANT_ARCHITECTURE.md) | Mos; cardinality PROPOSED |
| BIZ-009 — uz/en/ru, yashirin yangi o‘zbek alifbosi | [Localization §1, §7](../modules/LOCALIZATION_ARCHITECTURE.md) | Mos; locale IDs PROPOSED |
| BIZ-010 — UX asosiy raqobat ustunligi, murakkab imkoniyat/sodda jarayon | [UX §1, §3–11](../modules/UX_ARCHITECTURE.md) | Yo‘nalish mos; UX acceptance sinovsiz |

**Minus balans xulosasi:** [Billing §13](../modules/BILLING_ARCHITECTURE.md), [ADR-BILL-006](../adr/ADR-BILL-006.md), [Backend negative-month rules](../modules/BACKEND_ARCHITECTURE.md) bir kalendar oy haqiqiy xarajatni hisoblashni saqlaydi. Hard prepaid stop yoki forecast ceilingni APPROVED qoida sifatida kiritgan tasdiqlangan qaror aniqlanmadi. Lekin period anchor, timezone, delayed events, refill, recurring negative cycle, post-expiry service va settlement hali OPEN. Hech biri auditor tomonidan ixtiyoriy hal qilinmaydi.

**Texnik status:** Go — [ADR-BE-001](../adr/ADR-BE-001.md); PostgreSQL — [ADR-DB-001](../adr/ADR-DB-001.md); modular monolith — [ADR-003](../adr/ADR-003-modular-monolith.md) va [ADR-BE-002](../adr/ADR-BE-002.md); shared schema/RLS — [ADR-MT-001](../adr/ADR-MT-001.md); sharding/dedicated — [ADR-DB-008](../adr/ADR-DB-008.md), [ADR-INFRA-005](../adr/ADR-INFRA-005.md). Barchasi PROPOSED. Next.js va Flutter ham [Master §7](../modules/MASTER_ARCHITECTURE.md) va tegishli UX/Mobile hujjatlarida tavsiya, APPROVED emas.

**FIN-001:** [FIN-001](../adr/FIN-001.md) hujjatida APPROVED (business/domain boundary requirement) yozilgan, [Finance](../modules/FINANCE_ARCHITECTURE.md) ham shu ajratishni “APPROVED requirement” deydi; lekin foydalanuvchining ushbu alohida ADR uchun original tasdiq dalili repositoryda ko‘rsatilmagan. Audit buni “approval provenance unresolved” deb belgilaydi; hozirgi ADR statusini o‘zgartirmaydi.

---

# D. Conflict Register

**Rasmiy sonlar:** 13 topilma; **8 HIGH**, **5 MEDIUM**, **0 isbotlangan CRITICAL**. PROPOSED nomuvofiqliklar APPROVED qarorlar to‘qnashuvi sifatida talqin etilmaydi.

## AR-001 — Billing wallet ownership

- **Severity:** HIGH. **Tur:** CONFIRMED PROPOSAL MISMATCH.
- **Modullar:** 01 Billing ↔ 02 Multi-Tenant ↔ 06 Finance.
- **Hujjat/ADR:** [ADR-BILL-001](../adr/ADR-BILL-001.md); [Billing §3](../modules/BILLING_ARCHITECTURE.md); [Multi-Tenant §3, §10](../modules/MULTI_TENANT_ARCHITECTURE.md); [ADR-MT-002](../adr/ADR-MT-002.md).
- **Muammo:** Billing kompaniya billing accounti uchun yagona wallet va filial kesimida attribution taklif qiladi; Multi-Tenant bir tenantda bir nechta kompaniya ehtimolini saqlaydi, BillingAccount = Tenantni tasdiqlamaydi. Bular yakuniy qarorlar emas, lekin bir xillik yo‘q.
- **Xavf:** wallet aralashuvi, noto‘g‘ri seat/usage attribution, to‘lov, qarzdorlik va kompaniya chegarasi xatosi.
- **Yechim taklifi:** Tenant–Company–LegalEntity–BillingAccount–Wallet cardinality, lifecycle, payer, branch attribution, debt owner va migratsiya kontraktini yagona diagramma bilan belgilash.
- **Tuzatish joyi:** 01 + 02 + 06. **Foydalanuvchi tasdig‘i:** ha.

## AR-002 — Product va Catalog domain ownership

- **Severity:** HIGH. **Tur:** CONFIRMED PROPOSAL MISMATCH.
- **Modullar:** 04 Backend ↔ 05 Sales & Inventory ↔ 07 CRM ↔ 00 Master.
- **Hujjat/ADR:** [ADR-CRM-001](../adr/ADR-CRM-001-domain-ownership.md); [Backend §3](../modules/BACKEND_ARCHITECTURE.md); [Sales — Modullar va chegaralar](../modules/SALES_INVENTORY.md).
- **Muammo:** CRM ADR “Sales owns products, orders, inventory and fulfillment” deydi; Backend/Sales arxitekturasi Catalog, Sales, Inventory, Purchasing, Pricingni alohida bounded context sifatida ajratadi.
- **Xavf:** SKU/Product uchun ikki mutatsiya egasi, narx va o‘lchov birliklari drift, noto‘g‘ri import va chalkash API.
- **Yechim:** Product, SKU, Catalog, Pricing, Order, Stock, Purchasing uchun owner va public contracts matrixni birlashtirish; boshqa modul private table yozmasin.
- **Tuzatish joyi:** 00 + 04 + 05 + 07. **Tasdiq:** ha.

## AR-003 — Boshlang‘ich PostgreSQL HA

- **Severity:** MEDIUM. **Tur:** CONFIRMED PROPOSAL MISMATCH.
- **Modullar:** 03 Database ↔ 11 Infrastructure.
- **Hujjat/ADR:** [Database §9](../modules/DATABASE_ARCHITECTURE.md); [Infrastructure §7](../modules/INFRA_ARCHITECTURE.md); [ADR-DB-007](../adr/ADR-DB-007.md); [ADR-INFRA-002](../adr/ADR-INFRA-002.md).
- **Muammo:** Database boshlang‘ich primary+streaming standby (HA) deydi; Infrastructure Stage A single primary+PITR, Stage B standby deydi.
- **Xavf:** real availability va deployment cost assumptionlari chalkashishi, boshida HA deb noto‘g‘ri e'lon qilish.
- **Yechim:** Production bosqichi, SLO, RPO/RTO va narxga qarab bir initial topologiya, standby/failover bosqichi va recovery runbookni belgilash.
- **Tuzatish joyi:** 03 + 11. **Tasdiq:** ha.

## AR-004 — FIN-001 foydalanuvchi tasdig‘i manbasi

- **Severity:** HIGH. **Tur:** APPROVAL PROVENANCE GAP; biznes talabga qarshi isbotlangan ziddiyat emas.
- **Modullar:** 06 Finance ↔ 00 Master ↔ 01 Billing.
- **Hujjat/ADR:** [FIN-001](../adr/FIN-001.md); [FINANCE_ARCHITECTURE.md — Domain boundaries](../modules/FINANCE_ARCHITECTURE.md); [Master §1](../modules/MASTER_ARCHITECTURE.md).
- **Muammo:** ADR ichida APPROVED yozilgan; original foydalanuvchi tasdig‘iga aniqlanadigan GitHub manba yo‘q.
- **Xavf:** mustaqil tasdiqlanmagan domain qarorini Codex qat’iy approved implementatsiya buyrug‘i deb olishi.
- **Yechim:** explicit approval dalilini bog‘lash yoki foydalanuvchi bilan statusni qayta ko‘rib chiqish; **hozircha FIN-001 statusiga tegmaslik**.
- **Tuzatish joyi:** 06 + 00. **Tasdiq:** ha yoki mustaqil tasdiq dalili.

## AR-005 — SalesDocument posting va stock/GL trigger

- **Severity:** HIGH. **Tur:** CONTRACT AMBIGUITY.
- **Modullar:** 03 Database ↔ 05 Sales & Inventory ↔ 06 Finance ↔ 04 Backend.
- **Hujjat/ADR:** [Database §6](../modules/DATABASE_ARCHITECTURE.md); [Sales §1, §4, §10](../modules/SALES_INVENTORY.md); [ADR-DB-004](../adr/ADR-DB-004.md).
- **Muammo:** Database “SalesDocument posting + StockMovement + Finance journal posting”ni bitta transaction sifatida ifodalaydi. Sales order confirmation physical stock kamaytirmasligi, dispatch kamaytirishi aytilgan. SalesDocument umumiy tur bo‘lishi mumkin — bu tasdiqlangan biznes ziddiyati emas, balki transition nomi noaniqligi.
- **Xavf:** confirm va dispatchda ikki marta stock/GL effect yoki umuman yo‘q effect.
- **Yechim:** Order confirmation, reserve, dispatch, goods receipt, invoice, payment, GL posting, reversal uchun transition-to-effect matrix; atomic vs async boundaries, idempotency key, source-of-truth.
- **Tuzatish joyi:** 05 + 06 + 03 + 04. **Tasdiq:** ha.

## AR-006 — Mobile ADR-MOB-004 va ADR-MOB-009 yo‘qligi

- **Severity:** MEDIUM. **Tur:** CONFIRMED DOCUMENTATION GAP.
- **Modullar:** 09 Mobile & Offline ↔ 10 Security ↔ 04 Backend.
- **Manba:** [Mobile — ADR registry](../modules/MOBILE_ARCHITECTURE.md#adr-registry); baseline [docs/adr tree](https://github.com/yakhyodev/erp-saas-architecture/tree/016573c8426ed3ea3b1eacbd0dc81adb67861220/docs/adr).
- **Muammo:** ADR-MOB-004 “Conflict resolution and idempotency” va ADR-MOB-009 “Mobile authentication and device security” indeksda bor, real fayllari main’da yo‘q.
- **Xavf:** offline retry, revocation va device security uchun alohida tekshiriladigan qaror yo‘q.
- **Yechim:** ikkita PROPOSED ADR yaratish, alternatives, threats, sync-state contract va cross-module acceptance sinovlarini qo‘shish.
- **Tuzatish joyi:** 09 + 10 + 04. **Tasdiq:** taklifni approved qilish uchun ha.

## AR-007 — Umumiy arxitektura hujjatlari mavjud emas

- **Severity:** HIGH. **Tur:** CONFIRMED DOCUMENTATION GAP.
- **Modullar:** 00–13 barcha; 14 Review.
- **Manba:** [main tree baseline](https://github.com/yakhyodev/erp-saas-architecture/tree/016573c8426ed3ea3b1eacbd0dc81adb67861220); [Master §13](../modules/MASTER_ARCHITECTURE.md).
- **Muammo:** REQUIREMENTS.md, DECISIONS.md, MODULE_REGISTRY.md, OPEN_QUESTIONS.md, CHANGELOG.md yo‘q; Master faqat PROPOSED docs/modules ichida.
- **Xavf:** Codex modul taklifini butun tizim APPROVED qarori deb olishi, qaror-versiya va dependency drift.
- **Yechim:** birlamchi global reyestrlar, source links, ADR statuses va approval provenance; tasdiqsiz taklif APPROVEDga ko‘chmasin.
- **Tuzatish joyi:** 00 + 14. **Tasdiq:** fayllar mazmuni va yangi qarorlar uchun ha.

## AR-008 — Counterparty canonical owner va payer/sale-point

- **Severity:** HIGH. **Tur:** UNRESOLVED OWNERSHIP.
- **Modullar:** 07 CRM ↔ 05 Sales ↔ 06 Finance ↔ 12 Integrations.
- **Manba:** [CRM §2](../modules/CRM_DISTRIBUTION.md); [Database §4.2](../modules/DATABASE_ARCHITECTURE.md); [ADR-INT-005](../adr/ADR-INT-005.md); [ADR-CRM-001](../adr/ADR-CRM-001-domain-ownership.md).
- **Muammo:** CRM Counterparty profile bilan ishlaydi, canonical Party CRMmi yoki shared Partymi — ochiq; Database Counterpartyni Sales logical ERDga joylashtiradi; external contragent modelida payer va sale point ajratilishi mumkin.
- **Xavf:** dublikat shaxs/yuridik shaxslar, noto‘g‘ri AR/debt owner, to‘lov xatolari.
- **Yechim:** Party, Counterparty, Customer, Supplier, Outlet, Payer, ReceivableOwner va merge-history authoritative contracti.
- **Tuzatish joyi:** 07 + 06 + 05 + 12. **Tasdiq:** ha.

## AR-009 — Offline kechikkan Usage va bir kalendar oy

- **Severity:** HIGH. **Tur:** UNRESOLVED BUSINESS RULE.
- **Modullar:** 01 Billing ↔ 09 Mobile ↔ 04 Backend.
- **Manba:** [Billing §5, §13](../modules/BILLING_ARCHITECTURE.md); [Mobile — Stock and billing](../modules/MOBILE_ARCHITECTURE.md); [ADR-BILL-006](../adr/ADR-BILL-006.md); [ADR-MOB-008](../adr/ADR-MOB-008.md).
- **Muammo:** offline hodisa oyning oxirida sodir bo‘lib keyingi oyda kelishi mumkin. Event occurrence, server receipt, rating, ledger posting va billing period qay tarzda bog‘lanishi hal qilinmagan.
- **Xavf:** noto‘g‘ri davr, invoice/discount va negative balance, duplicate correction.
- **Yechim:** occurred_at, received_at, rated_at, posted_at, effective business/billing period, late-arrival closure va compensation policy; sun'iy credit ceiling kiritmaslik.
- **Tuzatish joyi:** 01 + 09 + 04. **Tasdiq:** ha.

## AR-010 — Chargeable document lifecycle

- **Severity:** HIGH. **Tur:** UNRESOLVED BUSINESS RULE.
- **Modullar:** 01 Billing ↔ 05 Sales ↔ 12 Integrations.
- **Manba:** [Billing §7, §16](../modules/BILLING_ARCHITECTURE.md); [Sales — Document workflow](../modules/SALES_INVENTORY.md); [ADR-BILL-003](../adr/ADR-BILL-003.md).
- **Muammo:** hujjatga asosiy narx approved, ammo qaysi hujjat turi/state billable, qanday amendment/cancel/return/import hisoblanishi aniqlanmagan.
- **Xavf:** double charge yoki missing charge.
- **Yechim:** document type × lifecycle transition × billable occurrence × reversal/correction × imported-history matrix; source idempotency.
- **Tuzatish joyi:** 01 + 05 + 12. **Tasdiq:** ha.

## AR-011 — SalesDoc dalil darajasi

- **Severity:** MEDIUM. **Tur:** DOCUMENTATION AMBIGUITY.
- **Modullar:** 07 CRM ↔ 12 Integrations.
- **Manba:** [ADR-CRM-003](../adr/ADR-CRM-003-salesdoc-adapter.md); [Integration — Evidence classification](../modules/INTEGRATION_ARCHITECTURE.md).
- **Muammo:** CRM ADR live API contract verifikatsiyasi ochiqligini yozadi; Integration protokolning hujjatdagi ayrim xususiyatlarini VERIFIED_DOC deb beradi. Bu ikki bayonot mantiqan mos bo‘lishi mumkin, ammo termin noto‘g‘ri talqin qilinishi mumkin.
- **Xavf:** public API documentation production live behavior deb qabul qilinishi.
- **Yechim:** DOCUMENTED, LIVE_VERIFIED, INFERRED, UNVERIFIED semantikasini bir xil belgilash; import/outbound gate oldidan live/sandbox test dalillari.
- **Tuzatish joyi:** 12 + 07. **Tasdiq:** kontraktni final qilish uchun ha.

## AR-012 — UX acceptance va top-20 task registry

- **Severity:** MEDIUM. **Tur:** REQUIREMENTS GAP.
- **Modullar:** 08 UX ↔ 00 Master ↔ barcha biznes modullar.
- **Manba:** [UX §1, §3, §11, §13](../modules/UX_ARCHITECTURE.md); [ADR-UX-004](../adr/ADR-UX-004.md).
- **Muammo:** task-first, global search, guided workflow va metrikalar yozilgan; tasdiqlangan top-20 task, barcha domenlar workflow registry, role/baseline va real usability test hali yo‘q.
- **Xavf:** chiroyli, ammo amaliyotda murakkab navigatsiya; acceptance mezonlari bo‘lmagani uchun Codex noto‘g‘ri flow yarata oladi.
- **Yechim:** top-20 task matrix, persona/permission mapping, domain state-to-UX states, 2–3 action discoverability testi va accessibility/performance benchmark.
- **Tuzatish joyi:** 08 + 00 + tegishli biznes modullar. **Tasdiq:** ha.

## AR-013 — Membership va permission ownership

- **Severity:** MEDIUM. **Tur:** BOUNDARY AMBIGUITY.
- **Modullar:** 02 Multi-Tenant ↔ 04 Backend ↔ 10 Security.
- **Manba:** [Multi-Tenant §2–4](../modules/MULTI_TENANT_ARCHITECTURE.md); [Backend §3](../modules/BACKEND_ARCHITECTURE.md); [ADR-SEC-002](../adr/ADR-SEC-002.md).
- **Muammo:** Tenant modul membership reference/lifecycle bilan ishlaydi; Backend Identity ownership qatorida membership/role association bor; Security permission semantics egasi. Ularning yagona source-of-truth va revoke contracti aniqlashmagan.
- **Xavf:** stale grant, privilege escalation, tenant switchdan keyingi eskirgan ruxsat.
- **Yechim:** Identity, Membership, RoleAssignment, PermissionEvaluationning bitta canonical ownership xaritasi va revocation/event propagation contracti.
- **Tuzatish joyi:** 02 + 04 + 10. **Tasdiq:** ha.

### D1. Severity yakuniy hisobi

- **HIGH (8):** AR-001, AR-002, AR-004, AR-005, AR-007, AR-008, AR-009, AR-010.
- **MEDIUM (5):** AR-003, AR-006, AR-011, AR-012, AR-013.
- **Tasdiqlangan CRITICAL (0):** tasdiqlangan biznes qoidasiga zid APPROVED arxitektura qarori yoki real production buzilishi isbotlanmadi.
- **LOW (0):** bu reyestrda alohida LOW topilma ochilmadi.

---

# E. Missing Requirements — MR-01…MR-20

Quyidagilar “yangi tasdiqlangan talab” emas, OPEN_QUESTIONS.md va tegishli modul ADRlari uchun **PROPOSED/OPEN** konsolidatsiya. Modul hujjatidagi savollar umumiy reyestrga kiritilmagan.

| ID | Ochiq talab yoki aniqlashtirish | Egasi | Severity/ustuvorlik |
|---|---|---|---|
| MR-01 | Tenant, Company, LegalEntity va BillingAccount cardinality, debt/Wallet ownership | 02, 01, 06 | HIGH |
| MR-02 | Negative balance bir kalendar oyining start/end, timezone, refill, expiry, recovery lifecycle | 01 | HIGH |
| MR-03 | Seat statuslari, billing davri, invitation/suspension, prorating, billable membership | 01, 10 | HIGH |
| MR-04 | Billable hujjat turi/lifecycle, base price, hajm metrikasi, threshold va resurs birliklari | 01, 05 | HIGH |
| MR-05 | Progressive tierlar, enterprise override precedence, stacking, rounding, pricing versions | 01 | HIGH |
| MR-06 | Finance jurisdiction, tax, legal entity, functional currency, fiscal close, accounting policy | 06 | HIGH |
| MR-07 | Sales, Inventory va Finance transition→effect, posting, reversal va atomicity boundary | 05, 06, 03 | HIGH |
| MR-08 | Negative stock, reservation trigger/TTL, inventory valuation, lot/serial, stock-count cutoff | 05 | HIGH |
| MR-09 | Party/Customer/Supplier/Payer/Outlet authoritative ownership, dedupe va merge | 07, 05, 06 | HIGH |
| MR-10 | Canonical API/event schema, version, retry, idempotency, ordering, deprecation, retention | 04, 12 | HIGH |
| MR-11 | Shared schema/RLS final approval, privileged bypass, tenant placement/migration/restore | 02, 03, 10 | HIGH |
| MR-12 | Mobile sync conflict matrix, offline session TTL, local revoke, late billing events | 09, 04 | HIGH |
| MR-13 | GPS lawful basis, consent/notice, media/PII retention, deletion, evidence export | 07, 09, 10 | HIGH |
| MR-14 | Production SLO, RPO/RTO, acceptable downtime, backup/object retention, DR owner | 11, 03 | HIGH |
| MR-15 | Peak load, largest tenant, p95/p99, connection/IO budget, cost/capacity envelope | 11, 03, 04 | MEDIUM |
| MR-16 | MVP top-20 task, persona/role mapping, UX workflow acceptance, user testing | 08, 00 | HIGH |
| MR-17 | Global search authorization, counts/suggestions/recents, index/reindex semantics | 08, 10, 03, 13 | HIGH |
| MR-18 | HR, Manufacturing, Reporting, Purchasing va boshqa optional domenlarning MVP scope/owner | 00, 05 | MEDIUM |
| MR-19 | SalesDoc public docs vs live behavior, outbound approval, migration mapping, cutover | 12 | HIGH |
| MR-20 | Accessibility baseline, device/network/locale profile, end-to-end UX performance | 08, 09, 13 | MEDIUM |

Manbalar: [Billing §16](../modules/BILLING_ARCHITECTURE.md), [Database §14](../modules/DATABASE_ARCHITECTURE.md), [Sales OPEN QUESTIONS](../modules/SALES_INVENTORY.md), [CRM §8](../modules/CRM_DISTRIBUTION.md), [Backend open questions](../modules/BACKEND_ARCHITECTURE.md), [Infra §15](../modules/INFRA_ARCHITECTURE.md), [Security §12](../modules/SECURITY_ARCHITECTURE.md), [UX §13](../modules/UX_ARCHITECTURE.md), [Localization §11](../modules/LOCALIZATION_ARCHITECTURE.md).

**Tavsiyaning chegarasi:** kamchiliklar implementation oldidan tekshirilishi kerak, lekin har biri majburiy MVP funksiyasi deb tasdiqlanmagan. Ayniqsa HR, Manufacturing, fleet, advanced lot/FEFO, dedicated DB, qo‘shimcha locale va kuchli infratuzilma to‘plamlari faqat scope qaroridan keyin implementatsiya qilinadi.

---

# F. UX/UI Quality Gate

**Ustuvor prinsip:** murakkab ERP funksiyalari foydalanuvchiga oson topiladigan, mantiqan tushunarli va tez bajariladigan ish oqimlari ko‘rinishida yetkazilishi kerak. Ko‘rinishning chiroyli bo‘lishi yetarli emas. Bu foydalanuvchi tasdiqlagan strategik biznes talabidir; quyidagi aniq raqamli mezonlar esa PROPOSED.

## F1. Mavjud kuchli arxitektura komponentlari

[UX_ARCHITECTURE.md §3–10](../modules/UX_ARCHITECTURE.md) quyidagilarni PROPOSED sifatida aniq tasvirlaydi:

- Task-first global shell, role dashboard, kontekstual tezkor amallar va barcha funksiyalar katalogi.
- UX permission’ni ko‘rsatadi, backend authoritative authorizationni bajaradi. Company/branch switcher access huquqi emas.
- Yangi foydalanuvchi uchun “Sodda”, tajribali foydalanuvchi uchun “Kengaytirilgan” taqdimot; ikkisi ham ruxsat berilgan funksiyalarni yashirib yo‘qotmasligi kerak.
- Universal search: mahsulot, mijoz/counterparty, buyurtma, savdo hujjati, to‘lov, xodim. Natijalar, counts, recents, autocomplete ruxsat bilan cheklanadi.
- Sotuv, omborga qabul, qarzdorlikdan to‘lovga o‘tish kabi guided flow; server tasdiqlamaguncha UI operatsiyani posted deb da'vo qilmaydi.
- Katta jadvallar uchun server-side pagination/filter/sort, saved views, accessibility; barcha qatorni brauzerga yuklamaslik.
- Offline pending/rejected/conflict/accepted holatlarini foydalanuvchiga aniq ajratish; desktop/mobile bir xil semantik oqim.
- Keyboard, screen reader, focus, contrast, text expansion, uz/en/ru va hidden alphabetga mos layout.

Manbalar: [ADR-UX-001](../adr/ADR-UX-001.md)–[ADR-UX-006](../adr/ADR-UX-006.md), [UX_UI_ARCHITECTURE.md](../modules/UX_UI_ARCHITECTURE.md) (historical).

## F2. Quality Gate matrix

| UX mezoni | Mavjud dalil | Audit gate |
|---|---|---|
| Task-first navigation | Global shell va misol oqimlar | Shartli; top-20 list tasdiqlanmagan |
| Role-based UX | Persona/dashboard misollar | Shartli; real permission role taxonomy ochiq |
| Universal search | Federated/search provider konsepsiyasi | Shartli; auth/index contract ochiq |
| Progressive complexity | Sodda/Kengaytirilgan ko‘rinish | Shartli |
| Quick actions, keyboard | Palette va shortcut takliflari | Shartli |
| Yangi foydalanuvchi onboarding | Completion target taklifi | BLOCKED: pilot natijasi yo‘q |
| Professional productivity | Bulk action, shortcut, dense table | Shartli; usability test yo‘q |
| Workflow-first consistency | Sotuv/qabul/to‘lov flow namunalari | BLOCKED: barcha kritk domain flowlar tasdiqlanmagan |
| Desktop/mobile parity | Shared task taxonomy | Shartli |
| Accessibility | WCAG 2.2 AA audit target | Shartli, sertifikat yoki audit natijasi yo‘q |
| Localization | uz/en/ru va yashirin alifbo | Talab mos, layout QA kerak |
| UX performance | Proposal budgetlar | BLOCKED: benchmark va device/network profile yo‘q |

## F3. Majburiy workflow-UX savollari

Har bir kritik oqimda foydalanuvchi quyidagilarni aniq ko‘rishi/tushunishi kerak:

1. Qaysi tenant/company/branch va rol bilan ishlayotgani.
2. Qaysi biznes natija va hujjatga erishayotgani.
3. Qaysi ma'lumot va tasdiqlar majburiyligi hamda sababi.
4. Qaysi bosqich **draft**, **queued**, **pending**, **committed**, **rejected** yoki **conflict** ekanligi.
5. Retry yoki correction nima qilishi, qayta postingga olib kelmasligi.
6. Operatsiyaning stock, financial ledger, SaaS wallet yoki customer debtga real ta’siri.

## F4. Biznes workflow qabul sinovlari

| Workflow | Sinov mavzusi |
|---|---|
| Yangi sotuv | Customer/product lookup, price & stock, draft, confirm, dispatch, error path |
| Ombor qabul | Partial receive, quarantine, quantity mismatch, reconciliation |
| To‘lov/qarzdorlik | Payer/invoice, partial allocation, settlement, exception |
| Offline field agent | Visit/order pending, idempotent sync, rejected rights, conflict review |
| SaaS Billing | Kredit balansi, real minus sarf, to‘lov, tarix, “taxminiy limit”ni ko‘rsatmaslik |
| Admin/security | Membership, branch grants, audit va tenant context |
| Universal search | Unauthorized names/counts/recents/suggestions sizib chiqmasligi |

## F5. PROPOSED metrikalar, natija emas

[UX §1 va §11](../modules/UX_ARCHITECTURE.md), [historical UX performance budget](../modules/UX_UI_ARCHITECTURE.md):

- Top-20 vazifalarning kamida **90%**ini 2–3 navigatsion harakatda topish (form input va majburiy approval bosqichlari kirmaydi).
- Pilot sinovda asosiy vazifalarni **95%** muvaffaqiyat bilan bajarish maqsadi; yangi foydalanuvchi onboarding completion bo‘yicha alohida 90% taklifi bor.
- Typical connectivity’da umumiy search birinchi foydali natija va common navigation **p75 < 1 s** maqsadi (tasdiqlangan SLA emas).
- Historical proposal: visual acknowledgment ~100 ms, LCP p75 <= 2.5 s, INP <= 200 ms, CLS <= 0.1 (mos sahifalar va qurilma/tarmoqda); bu ko‘rsatkichlar measured natija emas.
- WCAG 2.2 AA audit maqsadi; keyboard-only, screen reader, zoom, long Russian text va reduced motion testlari.

**UX QUALITY GATE: NOT PASSED** — raqobat ustunligiga mos dizayn yo‘nalishi mavjud, ammo tasdiqlangan top-task, role/user journeys, real usability/performance evidence va qabul siyosati yetarli emas. Bu UX yomon degani emas, tayyorlik dalillari yetishmaganini bildiradi.

---

# G. Security & Performance Assessment

## G1. Security design review

[SECURITY_ARCHITECTURE.md](../modules/SECURITY_ARCHITECTURE.md), [ADR-SEC-001](../adr/ADR-SEC-001.md)–[ADR-SEC-010](../adr/ADR-SEC-010.md):

| Sohasi | PROPOSED nazorat | Hali tekshirilishi kerak |
|---|---|---|
| Authentication | Global identity, verified membership, MFA, revocable web/mobile sessions | Auth/session TTL, SSO/MFA policy |
| Authorization | Backend deny-by-default RBAC + branch/object/state ABAC | Canonical role/membership/grant ownership |
| Tenant isolation | App checks + composite tenant keys + conditional RLS | RLS role/pool, privileged bypass, negative tests |
| API | BOLA/IDOR checks, CSRF/CORS, signed webhook, rate limit, idempotency | Endpoint-by-endpoint test |
| Audit | Tamper-evident critical events, original actor/acting-as | Retention, legal hold, privileged operations |
| Encryption | TLS, backups/object/device encryption, KMS | Key lifecycle, provider, rotation |
| Support | Restricted audited break-glass and acting-as UI | Approval flow, scope/expiry |
| Mobile | Encrypted local DB, re-auth on sync | Offline revoke/remote wipe limits |
| Integration | Tenant-scoped secrets, egress controls, anti-replay | Provider-specific credential/security tests |
| Backup | Encrypted offsite copy, restore drills | Recovery authorization, cross-tenant isolation, retention |

**Release isolation test matrix (PROPOSED gate):** tenant A cannot read/write tenant B; branch rights checked; object detail/list/export/search/autocomplete/count/storage/CDN checked; pooled connection context cannot leak; worker/queue/job/cache scoped; membership revoke/tenant switch invalidates previous authority; support audited; offline replay revalidates permissions. Manbalar: [ADR-SEC-003](../adr/ADR-SEC-003.md), [ADR-SEC-004](../adr/ADR-SEC-004.md), [ADR-SEC-010](../adr/ADR-SEC-010.md), [Multi-Tenant §6–7](../modules/MULTI_TENANT_ARCHITECTURE.md).

**Xulosa:** security konsepsiyasi qatlamli va mantiqli; live security test, RLS policy yoki privilege configuration bo‘lmagani sabab productionga tayyor, deb baholab bo‘lmaydi.

## G2. Database, infrastructure va performance design

Manbalar: [Database §6–11](../modules/DATABASE_ARCHITECTURE.md), [Infrastructure §3–12](../modules/INFRA_ARCHITECTURE.md), [Backend §4, §7–11](../modules/BACKEND_ARCHITECTURE.md), [ADR-DB-006](../adr/ADR-DB-006.md), [ADR-INFRA-003](../adr/ADR-INFRA-003.md).

**Ijobiy:** stateless API, bounded connection pools, tenant-aware indexes, keyset pagination, ACID+outbox, independent workers, durable jobs, monitor first; Redis, Kafka, read replica, microservice, partition, sharding va Kubernetesni avtomatik birinchi bosqichga kiritmaslik resurs samaradorligiga mos. Single-host birinchi bosqich HA deb noto‘g‘ri e'lon qilinmasligi Infra’da ochiq aytilgan.

**Bottleneck xavflari:**

- Hot SKU va bitta warehouse/product uchun qattiq row locks, oversell va deadlocklar.
- Wallet/credit ledgerga parallel debit va idempotency dedupe contention.
- PostgreSQL primary write saturation, WAL/IO, connection pool queue va long-running reports.
- Worker polling, outbox backlog, scheduler duplicate runs va replay.
- Mobil qurilmalar bir vaqtda qaytishi (sync burst), katta media upload, CRM GPS evidence.
- Universal search indexing/normalization va permission filteringning kechikishi.
- Noisy-neighbor tenant, analytics/export joblari asosiy OLTPdan resurs tortib olishi.
- Replica lagni billing, finance, stockda strong-consistency deb talqin qilish.
- Shared database ichida tenant-specific restore murakkabligi, backup/data residency.

**Performance gate / test plan:**

| Workload | Tekshiruv |
|---|---|
| Sales/Inventory | Concurrent reserve/post; one SKU contention, reversal va oversell-negative-stock policy |
| Billing | Parallel wallet debit/topup, event replay, month boundary, reconcile |
| Finance | Journal double-post prevention, period close, FX/rounding |
| PostgreSQL | Connections, lock wait, query p95/p99, WAL/IO, checkpoint, migration |
| API/search | Peak p95/p99, BOLA-safe search count/suggest, tenant fairness |
| Workers | Queue lag, crash+replay, poison/dead-letter, operator replay |
| Mobile | Offline delta burst, cursor expiry, media resumable upload, stale permission |
| Infrastructure | Peak/spike/soak/fault/failover, restore, tenant isolation, costs |

**RPO/RTO/SLO, budgets, TPS va instance sizes hali OPEN; audit o‘zboshimchalik bilan raqam tasdiqlamaydi.** Performance bahosi konseptual mos, lekin measured evidence talab qiladi.

---

# H. Codex Readiness Assessment

Ushbu holatlar **butun modulning implementation specification yetukligi** uchun. READY WITH CONDITIONS modulni kodga darhol topshirish huquqi emas.

| Chat / modul | Readiness | Shart/bloker |
|---|---|---|
| 00 Master | **BLOCKED** | Master/registry/decision provenance va domain ownership final emas |
| 01 Billing | **BLOCKED** | Wallet, prices, chargeable docs, negative period, seat/usage contracts ochiq |
| 02 Multi-Tenant | **BLOCKED** | Tenant/company/billing-account mapping, isolation ADR approval ochiq |
| 03 Database | **BLOCKED** | DB/HA, transaction and FK/isolation contract approval ochiq |
| 04 Backend | **BLOCKED** | Stack, identity owner, API/event contract, orchestration ochiq |
| 05 Sales & Inventory | **BLOCKED** | Posting trigger, stock policy, costing, correction ochiq |
| 06 Finance | **BLOCKED** | Jurisdiction/accounting policy va FIN-001 provenance |
| 07 CRM & Distribution | **BLOCKED** | Canonical Party, payer, route/delivery/KPI qoidalari |
| 08 UX/UI | **READY WITH CONDITIONS** | Top-20 tasks, user testing, roles, accessibility/performance |
| 09 Mobile & Offline | **BLOCKED** | ADR-MOB-004/009 mavjud emas, offline/auth/sync edge-case’lar |
| 10 Security | **BLOCKED** | Auth, permission, isolation approval va test matrix |
| 11 Infrastructure | **BLOCKED** | Initial HA, SLO, RPO/RTO, capacity va recovery |
| 12 Integrations | **BLOCKED** | API live evidence, ID mapping, migration/cutover, outbound policy |
| 13 Localization | **READY WITH CONDITIONS** | Locale/fallback/search/rollout technical approvals va QA |
| **Jami** | **12 BLOCKED; 2 READY WITH CONDITIONS; 0 READY** | **Codex implementation overall BLOCKED** |

### H1. Codex oldidan majburiy acceptance gates

1. **GATE-01 — Decision provenance:** biznes APPROVEDlar va PROPOSED ADRlar DECISIONS.md’da dalilli reyestrga kiritilgan; FIN-001 masalasi ochiq ko‘rsatilgan.
2. **GATE-02 — Ownership:** Tenant, Company, LegalEntity, Membership, BillingAccount, Wallet, Party, Catalog, Order, Inventory, Finance ownerlari kelishilgan.
3. **GATE-03 — Tenant & security:** app/DB/storage/cache/search/worker/backup isolation contract va tasdiqlash.
4. **GATE-04 — Billing:** price/rating, Seat, Business/Resource Usage, ledger/reconciliation, negative month va delayed-event/reversal semantikasi aniqlangan.
5. **GATE-05 — Transaction consistency:** Sales→Stock→Finance posting va Billing outbox effects/compensations diagrammalari.
6. **GATE-06 — API/offline/integration:** versioned commands/events, sync states, idempotency keys, replay/timeout, external IDs, cutover.
7. **GATE-07 — Infra/recovery:** boshlang‘ich topology, SLO/RPO/RTO, backup/restore/monitoring, load envelope.
8. **GATE-08 — UX/UI:** top-20 workflows, role-aware permissions, accessibility, localization va task/performance acceptance.
9. **GATE-09 — Documentation:** root/shared reyestrlar, ADR approval/provenance, open-questions closure, change history va qayta audit.

**Codex uchun qoidalar:** APPROVED biznes invariantlarini o‘zgartirma; PROPOSED stackni final tanlama; noaniq policy bo‘yicha ixtiyoriy default qabul qilma; tenant/security bypass yo‘q; billing va ERP finance balansini birlashtirma; e-commerce hujjat/moliya/ombor postingini double effect bilan yaratma; UI offline draftni server posted deb aytmasin; tasdiqsiz kod generatsiya qilma.

---

# I. Required Corrections

## I1. Qaysi chatga qaysi tuzatish yuboriladi

| Chat | Topshiriq | Tegishli topilmalar |
|---|---|---|
| **00 Master** | Canonical domain boundaries, general Master, Decisions/Requirements/ModuleRegistry/OpenQuestions/Changelog, ADR provenance rejasini konsolidatsiya qilish | AR-002, AR-004, AR-007, AR-012 |
| **01 Billing** | Wallet/account cardinality, negative month, seat meter, chargeable document, discount/pricing/reversal va late usage kontraktlari | AR-001, AR-009, AR-010 |
| **02 Multi-Tenant** | Tenant→Company→Branch, billing account, membership authoritative ownership, placement/isolation | AR-001, AR-013 |
| **03 Database** | HA initial topology Infra bilan birlashtirish; composite keys/RLS, posting transaction, partition/dedupe va recovery qoidalari | AR-003, AR-005 |
| **04 Backend** | Domain/public port ownership, commands/events, identity membership, transaction orchestration, idempotent outbox | AR-002, AR-005, AR-013 |
| **05 Sales & Inventory** | Catalog/Product ownership, transition→effect, reservation, valuation, stock count, reversal, billing trigger | AR-002, AR-005, AR-010 |
| **06 Finance** | FIN-001 approval provenance, Party/payer/debt, GL posting triggers, legal entity/currency/tax | AR-004, AR-005, AR-008 |
| **07 CRM & Distribution** | Counterparty canonical ownership, payer/outlet, sales/distribution boundary, SalesDoc evidence | AR-002, AR-008, AR-011 |
| **08 UX/UI** | Top-20 tasks, workflow-first acceptance, permissions-aware search, accessibility, productivity, user testing | AR-012 |
| **09 Mobile & Offline** | ADR-MOB-004 va ADR-MOB-009, sync conflict/idempotency, device revoke, late usage | AR-006, AR-009 |
| **10 Security** | Membership/grant ownership, auth/tenant isolation, RLS and worker/cache/storage tests, mobile revoke | AR-013, AR-006 |
| **11 Infrastructure** | Initial HA Database bilan, production SLA/SLO/RPO/RTO, restore drills, cost/peak capacity | AR-003 |
| **12 Integrations** | DOCUMENTED vs LIVE_VERIFIED, scoped ID mapping, Party/payer, imported docs billing, cutover | AR-008, AR-010, AR-011 |
| **13 Localization** | UX/search locale contract, original data, reversible hidden alphabet, Unicode testing va glossary | UX cross-module quality gate |
| **14 Review** | Correctionsdan keyin qayta cross-module audit, AR closure dalillari va Codex readiness | Barcha |

## I2. Tavsiya etilgan ketma-ketlik

**1-bosqich — Canonical ownership:** 00, 01, 02, 04, 05, 06, 07. Tenant, wallet, membership, Catalog va Party modeli birinchi hal qilinadi.

**2-bosqich — Transaction integrity:** 01, 03, 04, 05, 06, 09, 12. Hujjat statusidan stock/GL/billing effectgacha auditli transition matrix va replay qoidalari.

**3-bosqich — Security va scaling:** 02, 03, 09, 10, 11. Isolation, privileges, mobile session, DR, HA va benchmark.

**4-bosqich — UX quality:** 08 + barcha biznes modullar + 13. Top-20, onboarding, role, navigatsiya, search, offline status, accessibility, performance.

**5-bosqich — Documentation sync va qayta review:** 00 + 14. ADR approvals, open questions closure, master/requirements/registry/changelog, yangi audit baseline va final Codex readiness.

**Cheklov:** tegishli modul chatlari o‘z eski APPROVED qarorlarini foydalanuvchi ruxsatisiz o‘zgartirmaydi; yangi qarorlar PROPOSED bo‘lib turadi, tasdiqlangach alohida reyestrga kiritiladi. Review faqat tavsiya beradi, avtomatik implementatsiyaga vakolat bermaydi.

---

# J. Final Architecture Review Summary

## J1. Yakuniy baho

| O‘lcham | Natija |
|---|---|
| Tasdiqlangan biznes qoidalariga konseptual moslik | Umuman mos; isbotlangan approved-vs-approved ziddiyat topilmadi |
| Yagona arxitektura konsepsiyasi | Izchil asos mavjud, texnik status PROPOSED |
| Domain ownership | Tuzatish talab qilinadi |
| Database va transaction correctness | Yaxshi takliflar mavjud, posting boundaries/HA ochiq |
| Billing integrity | Idempotency/ledger yo‘nalishi mos, muhim billing edge-case’lar ochiq |
| Tenant/security | Qatlamli dizayn, approval va negative/security sinovlari yo‘q |
| Scaling/performance | Resource-efficient proposal, real sizing/benchmark va HA agreement yo‘q |
| UX/UI | Task-first konsepsiya kuchli, user task evidence va acceptance yetishmaydi |
| Localization | 3 til + hidden alphabet mos, technical details PROPOSED |
| ADR coverage | 90 mavjud, Mobile registrida 2 ta missing |
| Docs synchronization | Beshta umumiy hujjat yo‘q; Master modul proposed |
| Codex implementation | BLOCKED |

## J2. Eng ustuvor besh masala

1. **Tenant/BillingAccount/Wallet cardinality** — noto‘g‘ri mappingda seat/usage/balance/debt aralashadi.
2. **Canonical domain ownership** — Product/Catalog, Party/Counterparty/Payer, Membership/Permissions yagona bo‘lishi kerak.
3. **Business transaction posting** — Order, Inventory, Finance, Billing transition va event/reversal kontraktlari ikki marta iqtisodiy effectni oldini olishi shart.
4. **Minus balance / delayed usage** — bir kalendar oy real sarf, taxminiy ceiling yo‘qligi bilan barcha timezone/late-event/settlement edge-case mos kelishi shart.
5. **UX task readiness** — faqat yaxshi interfeys emas, sodda topiladigan va bajariladigan ish jarayonlari; acceptance real foydalanuvchi testlari bilan.

## J3. Yakuniy verdict

**ARCHITECTURE STATUS: REVIEWED — CORRECTIONS REQUIRED.**

Loyiha ko‘p kompaniyali mustaqil ERP SaaS sifatida qurishga asos bo‘la oladigan, yaxshi ajratilgan konseptual arxitekturaga ega. Ammo repositoryning audit baseline holati hali **FINAL APPROVED SYSTEM SPECIFICATION** emas; Codex uni approvalsiz implementatsiya manbasi deb qabul qilmasligi kerak.

- **Documentation:** 108 Markdown = 17 module + 90 ADR + README; 14 architecture directions.
- **Conflict Register:** 13 = 8 HIGH + 5 MEDIUM + 0 confirmed CRITICAL.
- **Missing Requirements:** MR-01…MR-20.
- **Codex readiness:** 12 BLOCKED + 2 READY WITH CONDITIONS + 0 READY.
- **FIN-001:** GitHubda APPROVED deb yozilgan, original user approval independently unproven; ADR statusiga bu review tegmaydi.
- **User-confirmed business rules:** o‘zgartirilmadi.
- **PROPOSED options:** o‘zaro farq approved architecture conflict sifatida ko‘rsatilmadi.
- **Next step:** AR-001…013 bo‘yicha module-owner corrections → ADR review/explicit approvals → unified shared docs → final re-review → keyingi bosqichga alohida ruxsat.

**Auditning harakat chegarasi:** ushbu hisobot dalillarni saqlash va tuzatishlarni belgilash uchun. Bu hisobot o‘z-o‘zidan yangi biznes/texnik qarorlarni APPROVED qilmaydi; boshqa modul/ADRlar va README o‘zgartirilmaydi.
