# MASTER ARCHITECTURE — ERP SaaS

- **Versiya:** 0.1
- **Holat:** PROPOSED (texnik arxitektura)
- **Yo‘nalish:** 01 — Master Architecture
- **Qamrov:** faqat ushbu chatda ishlab chiqilgan umumiy arxitektura
- **Kod:** yo‘q

> Bu modul hujjati loyiha ildizidagi `MASTER_ARCHITECTURE.md` faylini almashtirmaydi. Boshqa chatlarning yakuniy ADRlari bu hujjatga avtomatik tasdiqlangan tarzda kiritilmagan.

## 1. Maqsad va tizim chegaralari

Maqsad — Sales Doctor'ga mustaqil muqobil, minglab kompaniyalar uchun xavfsiz, tezkor va resurs samarador ERP SaaS. Platforma foydalanuvchi web/mobile mijozlari, API/application, biznes domenlari, platforma servis qatlamlari va infratuzilmadan iborat. Tashqi chegara: to‘lov provayderlari, banklar, SMS/email, marketplace va uchinchi tomon ERP/API tizimlari. Ularning ichki ishlashi ushbu tizim egaligiga kirmaydi.

### APPROVED biznes qarorlar

| ID | Qaror |
|---|---|
| BIZ-001 | Hybrid Prepaid SaaS |
| BIZ-002 | Arzon Seat + Business Usage + Resource Usage |
| BIZ-003 | Business Usage: hujjatga asosiy narx, hajm chegarasidan keyin qo‘shimcha kredit |
| BIZ-004 | 1 kredit = 10 so‘m |
| BIZ-005 | Progressiv hajm chegirmasi; enterprise uchun individual shartnoma |
| BIZ-006 | Balans tugagach, haqiqiy Seat va Usage xarajatlari bir kalendar oy davomida minusga yoziladi |
| BIZ-007 | Minus balans uchun taxminiy kredit limiti hisoblanmaydi |
| BIZ-008 | Kompaniyalar va filiallar yagona SaaS platformasida boshqariladi |
| BIZ-009 | uz/en/ru va yangi o‘zbek alifbosi uchun yashirin variant |

### PROPOSED texnik yo‘nalishlar

- Modular monolith; domenlarga ajratilgan ownership va aniq public contracts.
- Go backend; PostgreSQL transactional database; Next.js web; Flutter mobile. Barchasi hali tasdiqlanmagan.
- Tenant isolation uchun shared infrastructure bazasi, enterprise isolationga evolyutsiya; konkret DB isolation va RLS siyosati hali ochiq.
- Multi-server horizontal application scaling, keyinchalik partitioning/sharding va selective microservice extraction.

## 2. Domen xaritasi

| Domen | Javobgarlik | Asosiy bog‘liqlik |
|---|---|---|
| Tenant & Organization | Tenant, company, branch, membership konteksti | Security, Billing, barcha ERP modullar |
| Identity & Access | Authentication, authorization, permission enforcement | Tenant, API, UX |
| Subscription & Billing | Wallet, seat, usage, pricing, credit ledger, billing invoice | Tenant, usage producerlar, payment adapters |
| Sales & Purchase | Buyurtma, sotuv, xarid va hujjat lifecycle | Inventory, CRM, Finance |
| Inventory | Ombor, qoldiq, transfer, inventarizatsiya | Sales, Purchase, Finance |
| Finance & Accounting | Receivable, payable, cash, payment, business ledger | Sales, Purchase, Inventory |
| CRM & Distribution | Customer, agent, territory, route, visit, GPS, KPI | Sales, Mobile |
| Mobile & Offline | Field workflow, local queue, sync | CRM, Sales, Auth |
| Integrations | External adapterlar, migratsiya | Har bir tegishli domen |
| Reporting | Read models va analytics | Domenlar tomonidan berilgan ma'lumotlar |
| Localization | uz/en/ru, hidden Uzbek alphabet; translation va qidiruv normalizatsiyasi | Web, Mobile, reporting |
| Audit / Observability | Xavfsizlik auditlari, log, metrics, traces | Barcha modullar |

HR va Manufacturing kabi qo‘shimcha domenlar **OPEN QUESTION**; ular ushbu chatda tasdiqlanmagan.

## 3. Dependency map

```text
Web / Mobile / External API
           |
Identity + Tenant Context + Authorization
           |
      ERP application
    /       |        \
 Sales -- Inventory -- Finance
   |          |          |
 CRM ------ Reporting ---- Integration
           |
 Platform: Billing / Audit / Localization
           |
 DB / Queue / Object Storage / Monitoring
```

Diagramma mantiqiy yo‘nalish; modullar bir-birining private jadvallarini bevosita o‘qishi yoki yozishi ruxsat etilganini anglatmaydi. Hujjatlar va hodisalar uchun domain owner mas'ul. Atomic consistency talab qilinadigan workflow chegaralari modul spetsifikatsiyalarida belgilanishi kerak.

## 4. Application architecture

**PROPOSED:** dastlab modular monolith. Bitta deployable ichida alohida domain/application/infrastructure chegaralari; modul public interface yoki ichki application contract orqali murojaat; o‘zaro private data access taqiqlanadi. Business operation uchun transaction chegarasi aniq; async ishlar uchun outbox/idempotency yo‘nalishi baholanadi. Queue texnologiyasi hozircha tasdiqlanmagan.

## 5. Scaling evolution

1. Modular monolith, kuchli observability va indeks optimallashtirish.
2. Stateless application serverlar + load balancer; shared DB bottlenecklarni o‘lchash.
3. Background worker separation, read replica va ehtiyojga ko‘ra partitioning.
4. O‘lchov bilan oqlansa, modulni contract saqlagan holda selective microservicega ajratish.
5. Katta tenantlar uchun enterprise isolation yoki sharding: migratsiya, data residency, operatsion qiymat va SLA asosida alohida ADR.

Microservice extraction majburiy bosqich emas. Extraction qaysi moduldan boshlanishi **OPEN QUESTION**.

## 6. Multi-tenant va ma'lumot egaligi

**APPROVED:** kompaniya/filiallar yagona SaaS platformasida. **PROPOSED:** tenant context har bir sensitive business read/write va background jobda tekshiriladi; company/branch permission qo‘shimcha cheklov bo‘lishi mumkin; cross-tenant access default deny. Shared schema, separate schema, dedicated DB va RLSning aniq kombinatsiyasi OPEN QUESTION. Tenant ID har obyektga ko‘r-ko‘rona kiritilmaydi: global va tenant-scoped obyektlar ownership qoidasi bilan aniqlanadi.

## 7. Texnologiya qatlamlari

- **Backend PROPOSED:** Go, modular monolith, API contracts, worker va event processing.
- **Database PROPOSED:** PostgreSQL, ACID business transactions, tenant-aware keys/indexes, backups va restore-test.
- **Frontend PROPOSED:** Next.js, role-based UI, oddiy navigatsiya va auditga mos harakatlar.
- **Mobile PROPOSED:** Flutter, field workflows; offline conflict resolution hali alohida tasdiq talab qiladi.
- **Infra OPEN QUESTION:** cache/queue/object storage/HA/DRning aniq mahsuloti va SLOlari.

## 8. Billing boundary

**APPROVED:** Hybrid Prepaid, real Seat va Usage minus balans qoidalari, 1 kredit = 10 so‘m. **PROPOSED:** Billing mustaqil platform domaini; Sales/CRM/Inventory kabi biznes modullar billable usage factlarni kontrakt orqali beradi, tarif va chegirma hisobini Billing bajaradi. SaaS Billing ledger bilan tenantning biznes Accounting ledgeri bir xil emas. Negative balance holatidagi kirishni bloklash va abonent statusi bo‘yicha qoida hali alohida kelishilishi kerak; taxminiy limit joriy qilinmaydi.

## 9. Security

**PROPOSED:** identity/tenant/permission contextni markaziy usulda tekshirish; modulga tegishli biznes ruxsatlarini domain darajasida majburiy nazorat qilish; audit events, encryption in transit/at rest, secret management, rate limiting, least privilege. RLS va aniq RBAC/ABAC modeli SECURITY hamda MULTI_TENANT hujjatlariga bog‘liq va tasdiqlanmagan.

## 10. Localization

**APPROVED:** uz/en/ru va yashirin kelajakdagi o‘zbek alifbosi. **PROPOSED:** key-based translations, locale fallback policy, business terminlari katalogi, search normalization va UI/business data translationning ajratilishi. Yashirin locale ni tasodifan publicga chiqarish mumkin emas.

## 11. ADR registry (faqat ushbu chat)

| ADR | Mavzu | Status |
|---|---|---|
| ADR-001 | System boundaries | PROPOSED |
| ADR-002 | Domain ownership | PROPOSED |
| ADR-003 | Modular monolith | PROPOSED |
| ADR-004 | Scaling evolution | PROPOSED |
| ADR-005 | Tenant strategy | PROPOSED |
| ADR-006 | Database platform | PROPOSED |
| ADR-007 | Billing boundary | PROPOSED |
| ADR-008 | Security model | PROPOSED |
| ADR-009 | Localization model | PROPOSED |

ADR tafsilotlari `../adr/` da saqlanadi. Hozircha ushbu ADRlar **APPROVED emas**.

## 12. Open questions va konfliktlar

- Tenant vs company vs branch uchun universal ownership contract tasdiqlansin.
- Billing minus balansni qanday settlement qilish, bir kalendar oy boshlanish/yakun chegaralari va keyingi davr access siyosati aniqlansin. APPROVED qoidaga zid sun'iy limit qo‘yilmasin.
- Inventory va Finance transaction consistency va event ordering belgilansin.
- Tenant data RLS isolation, migration, backup/restore va enterprise dedicated deployment mexanizmlari tasdiqlansin.
- Offline conflict resolution, GPS privacy va image upload cheklovlari kelishilsin.
- HR/Manufacturing scope tasdiqlanmagani uchun delivery commitment emas.

## 13. Synchronization taklifi (shared fayllarga yozilmagan)

- `DECISIONS.md`: BIZ-001..009 approvedlarini qayd etish, TECH/ADR-001..009ni proposed ro‘yxatga olish.
- Root `MASTER_ARCHITECTURE.md`: ushbu modul arxitekturasiga havola berish, tasdiqlangan cross-module qarorlarni reviewdan keyin yig‘ish.
- `MODULE_REGISTRY.md`: domenlar va owner boundarylarni tegishli chatlar bilan solishtirib kiritish.
- `OPEN_QUESTIONS.md`: §12 savollarini ko‘rib chiqish.
- `CHANGELOG.md`: faqat merge/tasdiqdan so‘ng yozilgan o‘zgarishlarni qayd etish.

## 14. Codex architecture specification — kod emas

Codex implementatsiya boshlashdan oldin: (1) approved biznes qoidalarni o‘zgartirmasligi; (2) texnik takliflarni approvalsiz final qaror qilmasligi; (3) tenant/security enforcementni chetlab o‘tmasligi; (4) domain-owned write model chegaralarini saqlashi; (5) Billing va business Finance ledgerlarni ajratishi; (6) uz/en/ru va hidden locale talablarini ta'minlashi; (7) cross-module breaking changega ADR taklif qilishi; (8) microservice va shardingni o‘lchovlarsiz majburiy deb olmasligi kerak.

**Ushbu hujjat master arxitekturaning taklif versiyasidir; boshqa chatlarning qarorlari audit qilinmagan.**


## 15. Cross-module reconciliation pointer — 2026-10-11

The central registries [REQUIREMENTS.md](../../REQUIREMENTS.md), [DECISIONS.md](../../DECISIONS.md), [MODULE_REGISTRY.md](../../MODULE_REGISTRY.md), [OPEN_QUESTIONS.md](../../OPEN_QUESTIONS.md) and [CHANGELOG.md](../../CHANGELOG.md) are separate governance documents. Their initial version catalogs 13 user-supplied 14-audit gaps **AR-001..AR-013**, all **OPEN QUESTION** with review owners and proposed alternatives. **No cross-domain ownership, financial lifecycle, HA level, API verification level, billing charge trigger or membership authority is newly approved by this linkage.** Original decisions and historical ADR statuses remain unchanged; FIN-001's reported APPROVED label has missing independent approval provenance, tracked under AR-004. This section supplements, not supersedes, the historical master content above.


## 16. Round 2 authoritative coordination addendum (2026-10-12; PROPOSED)

**Evidence:** [Round 2 Architecture Review](../reviews/ARCHITECTURE_REVIEW_ROUND_2.md) based on historical audited commit `184b32f7b3dcc9ce8d329ca2dfb7895f06cae905`. Auditor verdict **BLOCKED**; nothing here overrides existing APPROVED business rules or FIN-001 provenance exception. Sections 1–15 above remain historical and their older tracking instructions must be interpreted together with v0.2 central registries, not as fresh status claims.

**Single cross-module dependency and ownership map:** the authoritative *coordination proposal* is maintained once in [MODULE_REGISTRY — Global dependency graph](../../MODULE_REGISTRY.md). This replaces diagram duplication for current design review; diagrams in §3 remain conceptual history. Candidate Catalog/Product/SKU, Pricing, Sales, Inventory, Purchasing, CRM Party/Outlet, Finance AR legal payer, TenantMembership, Identity/Credentials, RoleAssignment and PermissionEvaluation have distinct writers, while alternatives and contradictions remain OPEN.

**Corrections:** AR-006 Mobile ADR-004/009 exist, status **PROPOSED**, device/TTL/revocation policies remain OPEN. AR-007 all five root registry files exist; registry consistency work is not fully closed. [MR-01…20](../../OPEN_QUESTIONS.md) records evidence, owner, type, dependencies and next action; [97 ADR status/provenance index](../../DECISIONS.md) records 96 PROPOSED + FIN-001 tagged APPROVED but original approval source UNVERIFIED.

**Mandatory product invariant:** UX/UI must make *all ERP workflows* understandable, efficient and accessible. UX acceptance matrices are PROPOSED and NOT_TESTED, not measured performance claims. **MVP inclusion/exclusion for HR, Manufacturing, Purchasing, Reporting remains OPEN** (MR-18). No module mention is scope approval. Go/PostgreSQL/Next.js/Flutter/modular monolith/HA levels remain PROPOSED.

**Open conflicts not silently settled:** AR-001 wallet/cardinality; AR-002 Catalog scope; AR-004 FIN provenance; AR-005 DB §6 vs §17 transaction semantics; AR-008 Party/Payer; AR-009 late offline rating; AR-010 doc charge trigger; AR-013 TenantMembership writer conflict. These require owner/user decisions; do not initiate coding under Round 2 BLOCKED verdict.
