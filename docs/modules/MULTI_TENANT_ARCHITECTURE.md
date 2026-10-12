# Multi-Tenant Architecture

> Status: **PROPOSED** | Date: 2026-10-11 | Owner: Multi-Tenancy Architecture
> Ushbu hujjat arxitektura taklifidir; kod yoki yakuniy texnologiya tasdig‘i emas.
> ADRlar: [ADR-MT-001](../adr/ADR-MT-001.md), [ADR-MT-002](../adr/ADR-MT-002.md).

## 1. Qarorlarning maqomi

**APPROVED (oldingi biznes qarorlari):** ko‘p kompaniya/filialli yagona SaaS; Hybrid Prepaid SaaS; arzon Seat + Business Usage + Resource Usage; Business Usage uchun hujjat asosiy narxi va hajm chegarasidan keyingi qo‘shimcha kredit; 1 kredit = 10 so‘m; progressiv chegirma va enterprise individual shartnoma; balans tugaganda haqiqiy Seat va Usage xarajatlarini bir kalendar oy minusga yozish, taxminiy kredit limiti hisoblamaslik; uz/en/ru hamda yangi o‘zbek alifbosining yashirin varianti.

**PROPOSED:** Tenant → Company → Branch iyerarxiyasi; global User + tenant membership; tenant isolation boundary; shared PostgreSQL database/shared schema + RLS; placement registry; dedicated enterprise DB; tenant ID bo‘yicha sharding.

**NOT APPROVED:** Go, PostgreSQL, Next.js, Flutter yakuniy texnologik tanlov emas. **DEPRECATED/SUPERSEDED:** ushbu modulda hozir yo‘q.

## 2. Modul chegarasi

Modul tenant registry, lifecycle, company/branch relation contract, membership reference, active tenant context, isolation policy, tenant routing, placement va migration contract uchun javob beradi. Parol/MFA/sessiya autentifikatsiyasi Identity; permission katalogi va rollar Security; tariflar, seat metrigi, kredit/qarzdorlik Billing; yuridik accounting Finance moduliga tegishli.

Prinsiplar: isolation-first, default-deny, stable immutable tenant_id, bir global identity bilan ko‘p membership, biznes modullaridan physical DB placementni yashirish, tenantlararo joinlarni oddiy ERP transactionlardan chiqarish, tenant-aware idempotent billing eventlar.

## 3. Domen modeli (PROPOSED)

- **Tenant:** global immutable tenant_id; display_name; lifecycle status (provisioning/active/suspended/closing/closed); default_locale; timezone; placement_id; policy_version. Tenant yuridik kompaniyaning aynan o‘zi bo‘lishi shart emas.
- **Company:** company_id, tenant_id, legal_name, status, legal_metadata. Bir tenantda bir nechta company imkoniyati — tasdiqlanmagan taklif; kompaniyalararo hisob-kitob Finance bilan belgilanadi.
- **Branch:** branch_id, tenant_id, company_id, name, status va ixtiyoriy parent_branch_id. Branchning companyga tegishliligi serverda tekshiriladi; filial tree va inheritance hali OPEN.
- **User:** global user_id Identity domenida; global userning mavjudligi boshqa tenant ma’lumotiga kirish huquqini bermaydi.
- **TenantMembership:** membership_id, user_id, tenant_id, status, joined_at, revoked_at; tenant bo‘yicha mustaqil, revocationdan so‘ng so‘rov qayta tekshiriladi.
- **Assignments / Grants:** company/branch/resource scopega oid bog‘lanish; role/permission semantikasi Security domenida. Tenant ID barcha tenant-owned operatsion yozuvlarda majburiy.

## 4. Permission boundaries (PROPOSED)

Scope hierarchy: platform (faqat alohida service-admin) → tenant → company → branch → resource. Har so‘rov uchun authenticated user, faol tenant, aktiv membership, talab etilgan permission va tegishli company/branch ownership tekshiriladi. UI tenant selector xavfsizlik manbai emas. Tenant switch yangi server-verified context yaratadi va eski tenant cache/query state tozalanadi. Global support/break-glass access qat’iy audit, vaqt chegarasi va alohida Security ADRsiz ochilmaydi.

## 5. Database isolation variantlari

| Variant | Yutuq | Xavf / xarajat | Baholash |
|---|---|---|---|
| Shared DB / shared schema, faqat app filter | Arzon | App xatosi data leak keltiradi | Rad etish taklifi |
| Shared DB / shared schema + RLS | Arzon, migratsiya boshqaruvi sodda | Context, pool va privileged-role xavfi | **Boshlang‘ich PROPOSED** |
| Schema per tenant | Mantiqiy alohidalik | Minglab schema va migration murakkab | Muqobil |
| Dedicated DB per tenant | Kuchliroq isolation/restore | Resurs va operatsion xarajat katta | Enterprise PROPOSED |
| Hybrid pooled + dedicated | Tenantga mos joylashuv | Routing, move/cutover murakkab | Evolyutsion PROPOSED |

## 6. PostgreSQL RLS va connection safety (PROPOSED)

Har bir tenant-owned jadvalda tenant_id mavjud; ilova ishlatadigan DB role RLSni chetlab o‘tolmaydi; tenant identifikatori autentifikatsiya/member tekshiruvlaridan keyin serverda aniqlanadi va transaction scopega bog‘lanadi. Missing/mismatched tenant context → fail closed. Connection pool qayta foydalanishda context sizishi mumkin emas; background workerlar ham tenant contextni o‘rnatishi shart. Privileged migration/admin rollar va runtime rollar ajratiladi. Query, bulk import, upsert, report, audit, cache, search va file/object access uchun cross-tenant negative-testlar talab qilinadi. Company/branch authorization faqat tenant RLS bilan avtomatik hal bo‘lmaydi; server permission enforcement zarur.

## 7. Routing / serverlararo isolation (PROPOSED)

Control-plane **TenantRegistry**: tenant_id, lifecycle_status, placement_id, placement_epoch, routing_state, updated_at. **Placement**: placement_id, kind=pooled|dedicated, region, db_cluster_ref, secret_ref, state. Serverlar stateless tenant contextdan registry lookup qiladi; registry/cache stale bo‘lsa xavfsiz rad etadi. Placement epoch eski writerlarni fencing qiladi. Serverlararo event/queue/storage/cache namespace va authorization tenant-aware; secret va connection credentials ilova clientiga berilmaydi. Bir tenantning data joylashuvi almashganda tenant_id o‘zgarmaydi.

## 8. Boshlang‘ich, enterprise va migration strategiyasi

**Boshlang‘ich:** umumiy application deployment + pooled DB/shared schema/RLS, tenant-aware observability, quotas/backpressure va xavfsiz provisioning. **Enterprise:** compliance/yuk/contractga qarab dedicated DB yoki instance, lekin bu triggerlar hali tasdiqlanmagan. **Move lifecycle:** prereq/backup → target provisioning → copy/sync → verify → write fencing/cutover va placement epoch update → validation → old placementni retention qoidalari bo‘yicha tozalash. Dual write yoki CDCni default deb tasdiqlamaslik; cutover/rollback/RPO/RTO keyingi ADRda qaror qilinadi.

## 9. Sharding (PROPOSED)

Tenant_id — asosiy shard-placement kaliti, lekin hash(tid) kabi qattiq bog‘langan mapping o‘rniga tenant registry orqali shard assignment. Pooled tenantlar bir nechta shardga taqsimlanadi; rebalancing tenant-move mexanizmi orqali. Company/branch kesimida sharding tavsiya qilinmaydi, chunki tenant ichida cross-shard transaction xavfini oshiradi. Juda yirik tenant uchun dedicated placement; bitta tenantning ichki shardingi alohida kelajak ADR. Global IDs, tenant-aware migrations, restore, analytics va billing eventlari shardingga tayyor bo‘lishi kerak.

## 10. Billing integratsiyasi

Multi-Tenancy billable eventni tenant_id, company_id?, branch_id?, user_id?, membership_id?, event_id, idempotency_key, event_type, occurred_at, quantity, resource_dimension?, source_document_id?, schema_version bilan attributsiya qiladi. Pricing, seat counting, discount va minus-balans lifecycle faqat Billingga tegishli. **Billing account = tenant** deb tasdiqlanmagan; billing_account alohida bog‘lanish sifatida saqlansin. Migration paytida duplicate event idempotency bilan bloklanishi shart. Balans minusga o‘tganda suspension siyosati alohida kelishiladi; taxminiy credit limit kiritilmaydi.

## 11. Nofunksional qabul mezonlari

- Har bir tenant-owned resource boshqa tenantdan o‘qish/yozish imkonini bermaydi.
- Membership revoke va tenant switchdan so‘ng old context ishlamaydi.
- Runtime DB roli RLS bypass qilmaydi; pooled connection context leak testdan o‘tadi.
- Background jobs, storage, search, cache, logs va exports tenant-aware bo‘ladi.
- Tenant migrationda stable ID, fencing, verification, rollback plan, idempotent billing event talab etiladi.
- Tenant-specific restore, RPO/RTO, retention va performance SLO qiymatlari **OPEN**, o‘zboshimchalik bilan tasdiqlanmaydi.

## 12. Ochiq masalalar / potensial ziddiyatlar

1. Tenantda bir nechta mustaqil legal company: Master/Finance bilan kelishish.
2. Billing account mapping va Seat metriği: Billing bilan kelishish.
3. Branch nesting/inheritance: Finance/Sales/Inventory bilan kelishish.
4. Tenant selector/subdomain va token binding: Backend/Security/UX bilan kelishish.
5. Support break-glass va global admin: Security/Audit bilan kelishish.
6. Data residency, dedicated criteria, RPO/RTO, backup/restore: Infrastructure/Database.
7. Cutover/CDC/rollback va no-downtime: Backend/Database.
8. Tenant-specific search/storage/analytics placement: Infrastructure/Backend.
9. Minus-balans muddati tugaganda suspension policy: Billing/Security.

**Potensial ziddiyat:** boshqa modul Company = Tenant yoki BillingAccount = Tenant deb majburiy birlashtirsa ushbu PROPOSED modelga to‘g‘ri kelmasligi mumkin. Bu boshqa hujjat tasdiqlangan qarorini avtomatik bekor qilmaydi; cross-module review zarur.

## 13. Codex uchun specification (kelgusi bosqich, KOD YOZILMASIN)

Domenlar: TenantRegistry, Tenant, Company, Branch, Membership, ScopeGrant contract, Placement, TenantContext, MigrationState. Invariantlar: immutable tenant_id; every tenant-owned row carries tenant_id; no cross-tenant access; authorization is server-side; context fail-closed; runtime DB no bypass; idempotent billable events. API contractlar: tenant select/list (faqat membership), company/branch tree (scope bilan), membership lifecycle, registry lookup (internal only), migration/cutover (internal privileged only). Har APIga auth, tenant context, membership, scope, ownership, audit va pagination constraints qo‘llansin. Testlar: cross-tenant read/write, invalid company/branch, revoke, token switch, stale placement, RLS bypass, pooled connection reuse, worker/job context, cache/search/storage leakage, billing duplication, failed cutover/rollback. Migration va initial schema implementatsiyasi ADR approvaldan keyin.

## 14. Sinxronlashtirish takliflari (boshqa fayllar O‘ZGARTIRILMADI)

- `DECISIONS.md`: ADR-MT-001 va ADR-MT-002 ni PROPOSED reyestrga kiritish.
- `MASTER_ARCHITECTURE.md`: tenant boundary, membership, placement abstraction konseptlarini PROPOSED sifatida linklash.
- `REQUIREMENTS.md`: fail-closed isolation, tenant-aware events, negative isolation tests, migration/recovery NFR.
- `MODULE_REGISTRY.md`: Multi-Tenancy domeni va Security/Billing/Database/Infra dependencies.
- `OPEN_QUESTIONS.md`: yuqoridagi unresolved masalalar.
- `CHANGELOG.md`: 2026-10-11, proposed Multi-Tenancy + ADR-MT-001/002.

Related ADRs: [Isolation & Placement](../adr/ADR-MT-001.md) · [Hierarchy & Membership](../adr/ADR-MT-002.md).


---

## 15. Cross-module harmonization — AR-001 / MR-01 (2026-10-11, PROPOSED)

**Manbalar:** root `DECISIONS.md`, `REQUIREMENTS.md`, `MODULE_REGISTRY.md`, `OPEN_QUESTIONS.md`; `docs/reviews/ARCHITECTURE_REVIEW.md` faqat tarixiy audit baseline. **Status: OPEN QUESTION — quyidagi mapping hech biri APPROVED emas.**

### 15.1 Atamalar va relational boundaries

| Entity | Tavsiya etilgan ma'no (PROPOSED) | Identifikator / ownership | Cardinality bo‘yicha holat |
|---|---|---|---|
| Tenant | Application data isolation va placement boundary | tenant_id: Multi-Tenant | Tenant ↔ Company: OPEN |
| Company | ERP operatsion tashkilot va scope | company_id; tenant_id bilan domain binding: Multi-Tenant / Finance review | Tenant ↔ Company: 1:1 yoki 1:N OPEN |
| LegalEntity | Yuridik/moliya-huquqiy hisob birligi, Company bilan aynan sinonim emas | legal_entity_id: Finance master ownership kandidati | Company ↔ LegalEntity: 1:1 yoki N:1 yoki vaqtli mapping OPEN |
| Branch | Operatsion bo‘linma (Companyga qarashli) | branch_id, parent_company_id; scope mapping | Company ↔ Branch: 1:N taklifi; companysizlik filiallarga munosabat OPEN |
| BillingAccount | SaaS bo‘yicha majburiyat, invoice va tarifikatsiya targeti | billing_account_id: Billing authoritative | Tenant/Company ↔ BillingAccount: OPEN |
| Wallet | Kredit mablag‘lari va minus (qarz) ledger asosiy hisobi | wallet_id: Billing authoritative | BillingAccount ↔ Wallet: 1:1, 1:N, pooled OPEN |

**Cardinality alternativlari:**
- **A — sodda single-company:** 1 Tenant ↔ 1 Company ↔ 1 LegalEntity; Company 1:N Branch; Tenant 1:1 BillingAccount ↔ 1 Wallet. Migratsiya oson, lekin multi-company tenantni cheklaydi.
- **B — tenant-level consolidated:** Tenant 1:N Company, Company 1:1 LegalEntity (taxmin), Company 1:N Branch; Tenant 1:1 BillingAccount ↔ 1 Wallet. Filial va company usage bir hisobga yig‘iladi; legal entityga xarajat qayta ajratish savoli OPEN.
- **C — flexible payer:** Tenant 1:N Company; Company ↔ LegalEntity mustaqil aniq mapping; Tenant 1:N BillingAccount; har BillingAccount 1 yoki N Wallet (variant); Company/Branch → BillingAccount mapping effective-dated. Murakkabroq, enterprise uchun mos; debt ownership aniq kelishilmaguncha joriy etilmaydi.
- **D — per-company billing:** Tenant 1:N Company, har Company 1:1 BillingAccount; BillingAccount 1:1 Wallet; umumiy Seat va resurslarning taqsimlash formulasi OPEN.

**Invariant sifatida tavsiya (PROPOSED, tasdiqlanmagan cardinalityga bog‘liq emas):** referential tenant isolation; company/branchning tenantga tegishliligi tekshiriladi; legal entity va payer mapping vaqtli/auditlangan bo‘lishi mumkin; billing account/wallet IDlarni tenant IDdan chiqarib bo‘lmaydi; wallet balance hujjati Multi-Tenancy tomonidan yozilmaydi; bir usage faktidan bir nechta charge asossiz tug‘ilmaydi.

**Tasdiqlangan monetizatsiya cheklovi:** Hybrid Prepaid; Seat + Business Usage + Resource Usage; har hujjat asosiy narx + hajmdan keyingi qo‘shimcha kredit; 1 kredit = 10 UZS; progressiv/enterprise chegirma. Balans tugaganda **faqat haqiqiy Seat/Usage xarajatlari bir kalendar oy** minusga yoziladi, taxminiy limit yo‘q. Bu qoidalar wallet sonini, qarzning huquqiy egasini yoki month anchorini o‘z-o‘zidan tanlamaydi.

**AR-001 user/Billing/Finance decision gate:** yuridik to‘lovchi kim? Bitta tenant bir nechta yuridik shaxsni qamraydimi? Filial alohida payer bo‘la oladimi? Wallet per billing accountmi yoki pooled? Umumiy seat/resource usage qaysi payerga ajratiladi? Effective-dated payer switch va old qarzni kim ko‘taradi? Javob kelmaguncha barcha alternativlar OPEN.

### 15.2 Cross-company billing attribution (PROPOSED contract, pricing emas)

Operatsion faktlarda `tenant_id`, `source_company_id?`, `source_branch_id?`, `actor_user_id?`, `membership_id?` saqlanadi; server `branch.company_id` va `company.tenant_id` mosligini tekshiradi. `billable_subject_type/id` va `billing_account_id` faqat Billingdagi effective-dated, vakolatli mappingdan aniqlanadi (yoki mapping yo‘q bo‘lsa billing pending/reconciliationga tushadi); client tomonidan wallet/billing account tanlash trusted emas. Cross-company charge splitting/konsolidatsiya Billingda rate/allocate qilinadi; tenant boundary orqali cross-tenant pooling default emas. `source_document_id` qayta ishlatilganida idempotency saqlansin.

**Event proposal `UsageFactRecorded.v1`:** `event_id`, `schema_version`, `tenant_id`, `source_company_id?`, `source_branch_id?`, `source_entity_type`, `source_entity_id`, `actor_user_id?`, `membership_id?`, `occurred_at`, `recorded_at`, `quantity`, `unit`, `dimension`, `idempotency_key`, `correlation_id`, `causation_id?`, `reversal_of_event_id?`, `placement_epoch?`. Billing returns `Rating/ChargeAccepted|Rejected|Pending` correlated by event_id; event acceptance is not a guarantee that document is chargeable. Calendar-month attribution, backdated/offline eventlar, refund/adjustment, document charge trigger = Billing/Sales/Mobile OPEN (AR-009/010). Financial debit, debt period, discount and ledger are **Billing-owned**.

### 15.3 Contract proposals (implementatsiya emas)

| Interface | Owner (candidate) | Kirish/chiqish contract | Fail behavior |
|---|---|---|---|
| Identity.Authenticate | Identity/Security | identity credential → principal/user_id and verified session | unauthenticated |
| TenantMembership.Resolve | Multi-Tenant (candidate) | user_id + tenant_id → active membership/version | deny |
| Security.Evaluate | Security (candidate) | principal + membership + tenant/company/branch/resource + action + policy_version → allow/deny/reason | deny |
| Tenant.ResolveScope | Multi-Tenant | company_id/branch_id → same tenant and valid company relation | deny |
| TenantRegistry.ResolvePlacement | Multi-Tenant/Infra | tenant_id → placement + epoch, internal-only | fail closed |
| Billing.ResolvePayer | Billing | tenant, company, branch, event_time → billing_account_id / unresolved | pending, no guessed wallet |
| Billing.AcceptUsageFact | Billing | event envelope → accepted/pending/duplicate/rejected | idempotent |
| Database.TenantTransaction | DB/Backend | verified tenant context, transaction-scoped RLS identity | refuse missing/mismatch |

Contract versionlari, exact endpoint nomlari, delivery ordering va security token formatlari **PROPOSED**; Backend/API review zarur. UX tenant/company/branch switcher access manbai emas.

## 16. AR-013 — membership, credential va permission authority (PROPOSED)

**Ownership candidate (yakuniy APPROVED emas):**
1. **Identity/Auth** credentials, MFA, sessions va global principal `user_id`ning authoritative egasi.
2. **Multi-Tenant** Tenant/Company/Branch registry, user↔tenant membership lifecycle (membership_id, state, version)ning authoritative kandidati; `user_id` credential nusxasi emas.
3. **Security** RoleAssignments, grants/delegations, permission catalog/policy version va `PermissionEvaluation` authoritative kandidati. Har bir business module esa o‘z resursining ownership/scopeini authoritative aniqlab, evaluation natijasini **server-side** qo‘llaydi.
4. **Billing** SaaS Seat uchun qaysi membership/user hisoblanishi, proration va payer attribution owneri; membershipning mavjudligi **o‘zidan-o‘zi billable Seat deyilmaydi**.

**Lifecycle proposal:** invited → active → suspended → revoked; invited/expired userga access bermaydi; revoke terminal, re-join uchun yangi membership yoki explicit reactivation policy (OPEN). Membership aktiv bo‘lsa ham user session invalid, tenant suspended yoki grant yo‘q bo‘lsa deny. Membership/role/scope version o‘zgarsa cache/session authorization re-check qilinadi. Har tenant switchda new verified context; company/branch change hech qanday latent role escalationsiz. Pending invitation identity bo‘lmasa ham minimal PII va expiry bilan yuritiladi (Identity/Security review). Service principals, delegation, emergency support access va invitation accept identity-merge OPEN.

**Ambiguity (MODULE_REGISTRY conflict):** global registry Securityni “identity, membership, RBAC/ABAC” deb yozgan; old module “membership reference” deydi. Candidate A: Multi-Tenant membership state authority, Security permission authority (tavsiya); B: Security membership authority, Multi-Tenant tenancy graphning read modelini ushlab turadi. Ikki joyda writable membership source of truth bo‘lishi mumkin emas. AR-013 **OPEN** qoladi.

## 17. MR-11 — tenant isolation va privileged access (PROPOSED controls)

- Server-side principal→tenant membership→scope→resource ownership verification; permission check har request, job, stream, export, search, reportda majburiy. Deny-by-default, metadata yoki counts orqali cross-tenant leak ham taqiqlansin.
- Pooled PostgreSQL **shared schema + RLS** faqat PROPOSED. Tenant-owned jadvallarda tenant key va `USING`/`WITH CHECK` siyosati; runtime role table owner/superuser/BYPASSRLS bo‘lmaydi. Owner role ishlatilsa FORCE RLS risk review; migration/admin credentials runtime/APIga ochilmaydi.
- Connection pooling: tenant context faqat transaction-local va transaction oxirida yo‘q; invalid/missing context deny. Prepared statements, connection reuse, async jobs, retries, search/cache va bulk operations negative testdan o‘tishi kerak.
- Privileged break-glass: default disabled, explicit approved workflow, just-in-time, reason/ticket, narrow scope, audit trail, review/expiry; bypass RLS uchun application superuser mexanizmi yaratmaslik.
- Routing: registry source of truth + placement_epoch fencing; stale cache/unknown placement deny; pooled/dedicated movementda single-writer; snapshot verification, export/import, cutover, idempotency, rollback va backup/restore drills. Dedicated DB yoki per-tenant restore RPO/RTO default va'da emas.
- DB isolation, tenant mobility/restore, privileged access final approval faqat Database, Security, Infra va Master bilan ADR sign-offdan keyin; **MR-11 OPEN**.

## 18. Codex handoff gate (faqat specification, kod yozilmasin)

**BLOCKED until decisions:** AR-001 payer/cardinality/debt owner; AR-013 membership authority; MR-01 legal entity mapping; MR-11 DB isolation/migration/restore; AR-009/010 billing time/charge triggers. Kod generatsiyasi yoki migrationni bu taklif bilan boshlamaslik. Approvaldan keyingi acceptance scenarios: tenant X cannot inspect Y even through ID guessing/cache/search/report/object storage; membership revoke works across active sessions; role revocation forces deny; cross-company scopes enforce resource ownership; billable event unique/deduplicated; unresolved payer never auto-charges guessed wallet; pooled-connection reuse never leaks tenant context; placement switch does not duplicate ledger events; privileged access auditable; recovery does not restore data to wrong tenant.

Cross-module root registry synchronization **faqat taklif**: DECISIONS'da ADR-MT-001/002 PROPOSED; OPEN_QUESTIONS'da AR-001/013 va MR-01/11 unresolved gates; REQUIREMENTS'da isolation/event acceptance proposals; MODULE_REGISTRY'da authority alternatives. **Bu fayllar ushbu commit bilan o‘zgartirilmaydi.**

## 19. ROUND 2 CORRECTION — AR-001 / AR-013 / MR-01 / MR-11 (2026-10-12)

> **Status: PROPOSED contracts, OPEN business/ownership approvals.** Cross-checked on `main`: [Round 2 review](../reviews/ARCHITECTURE_REVIEW_ROUND_2.md), [Billing](BILLING_ARCHITECTURE.md) §3/§19, [Security](SECURITY_ARCHITECTURE.md) §17/§19, [Backend](BACKEND_ARCHITECTURE.md) §17, [Database](DATABASE_ARCHITECTURE.md) §5/§17, [MODULE_REGISTRY](../../MODULE_REGISTRY.md). This section qualifies earlier §3/§15–18, does not overwrite historical proposals. No test run or code.

### 19.1 Entity boundaries and ownership

| Entity | Authoritative candidate / meaning | Relation (OPEN alternatives) | Explicitly NOT |
|---|---|---|---|
| Tenant | Multi-Tenant; stable isolation, placement, application organization boundary | 1:1 or 1:N Company | automatically the contracting payer or legal debtor |
| Company | Multi-Tenant org hierarchy; business operating unit with stable company_id/tenant_id; Finance validates legal/accounting linkage | Tenant → Company 1:1 vs 1:N | necessarily one LegalEntity or one wallet |
| LegalEntity | Finance candidate for registered legal identity, tax/accounting obligations, effective relationships | Company ↔ LegalEntity 1:1 vs N:1 vs time-bounded relation | synonym of Tenant or automatically SaaS debtor |
| Branch | Multi-Tenant org hierarchy; tenant_id, company_id, branch_id; resource access boundary | Company → Branch 1:1 vs 1:N (hierarchical branches OPEN) | legal payer by default |
| BillingAccount | Billing authoritative contract/rating/invoice/liability account and payer assignment | Tenant/Company → BillingAccount N:1, 1:1 or effective-dated N:M | computed from tenant_id/company_id |
| Wallet | Billing authoritative credit and negative-balance ledger account | BillingAccount → Wallet 1:1 vs 1:N/pooled | ERP GL, company cash account, or Tenant ID |
| Payer/Debtor | Billing owns SaaS contracted payment obligation and mapped party/account; Finance/CRM Party legal identity mapping subject to AR-008 | may be third party or group payer (OPEN) | necessarily Tenant or branch |

Foreign relation candidate: `Branch(tenant_id, company_id)` must reference a Company of *same tenant*. A legal-entity/payer relationship is distinct from branch scope and must carry version/effective interval when changed. BillingAccount and Wallet are Billing-owned; do not give Tenant a writable balance or authority to select a wallet via untrusted client input. Distinguish source activity, billable subject, legal liability and settlement ownership.

### 19.2 Competing cardinality options (none APPROVED)

| Model | Tenant / Company / Branch | Billing arrangement | Benefit | Risks / approvals |
|---|---|---|---|---|
| A: single company | Tenant 1 Company; Company 1 Branch (only) | 1 BillingAccount, 1 Wallet | very small pilot simplicity | cannot support additional branches without later migration; not a universal MVP assumption |
| B: single legal company, multiple branches | Tenant 1 Company; Company 1:N Branch | one company-level BillingAccount / Wallet | branch operational attribution; simple payer | does not handle multi-company organization |
| C: multi-company, shared payer | Tenant 1:N Company; Company 1:N Branch | many companies map to one tenant-scoped BillingAccount, candidate 1 Wallet | consolidated cashflow, fewer topups | legal payer consent, expense allocation and debt holder must be explicit |
| D: company payer (Round 2 preferred candidate) | Tenant 1:N Company; Company 1:N Branch | each billable Company → separate BillingAccount and candidate Wallet 1:1 | traceable liability per company | Seat shared users, resource allocation and cross-company contracts remain OPEN |
| E: enterprise consolidated | Tenant 1:N Company and LegalEntity relations; branches under companies | contract-defined grouping, effective-dated N:M assignments to BillingAccount; wallet pooling only by explicit approval | flexible enterprise payer consolidation | complex transfers, invoice, tax, historical debt reconciliation, refunds |

Round 2 audit favors **D as a PROPOSED MVP candidate**, C/E as optional alternatives. It does NOT establish that Tenant = legal payer or debtor. Billing §3 says one account/wallet per company; Billing §19.1 clarifies this as **only option B** in Billing's own naming. This document uses A–E solely as local comparison labels; map semantically, not by letter. No cardinality, branch count, payer or wallet count is approved.

### 19.3 Seat / Usage attribution handoff to Billing, not rating

**Source evidence contract (PROPOSED, versioned):** `event_id`, `event_type`, `schema_version`, `tenant_id`, `source_company_id?`, `source_branch_id?`, `principal_user_id?`, `membership_id?`, `source_object_type/id`, `source_version?`, `occurred_at`, `recorded_at`, `producer`, `quantity`, `unit`, `idempotency_key`, `correlation_id`, `causation_id?`, `reverses_event_id?`. Company and branch ownership is resolved server-side at operation time; optional source company/branch are **not** authorization bypass or guessed payer. For a global seat or shared resource without one source company, send a **tenant-level attribution** and explicitly `source_company_id=null`; do not arbitrarily assign to first company. Audit effective membership/scope version.

**Seat:** membership lifecycle facts (e.g. `MembershipActivated.v1`, `MembershipSuspended.v1`, `MembershipRevoked.v1`) include tenant/user/membership/version, effective time and scoped company/branch grants if relevant. These are *inputs* not charge commands; Billing alone defines billable seat uniqueness (user vs membership vs company), snapshots, proration, eligibility and account allocation. A user with multiple branches/companies must not automatically incur repeated seats.

**Business/Resource Usage:** source document/resource fact includes source tenant, company, optional branch, validated authoritative scope, timestamps and stable event ID. Billing separately maps source attribution to effective-dated `billing_account_id` + `assignment_version` + `payer_party_id?` + `legal_entity_id?`; charge ledger preserves both **source attribution** and **liability snapshot**. Unresolved payer ⇒ pending reconciliation, **never guess wallet**. Split/consolidation and shared costs need Billing contract rule approval; source never calculates tariffs, volume tiers, discounts, 10 UZS conversion, wallet balance or negative debt.

**APPROVED invariants untouched:** Hybrid Prepaid; Seat+Business+Resource Usage, document base charge/threshold extras; 1 credit=10 UZS; progressive/enterprise discounts; only actual Seat/Usage debits continue in minus **one calendar month**, no estimated credit limit. Debt holder, expiry interpretation, payer switches, offline month attribution and charge trigger remain OPEN (Billing AR-009/010).

**API candidates:** `Tenant.ResolveOrganizationContext`, `Membership.GetEffectiveState`, `Security.EvaluateScope`, `Billing.ResolveAccountAt(tenant, source_company?, source_branch?, occurred_at)`, `Billing.RecordUsageFact`. Billing returns accepted/pending/duplicate/rejected with event correlation. Event ordering, retry, schema migration and exact endpoint naming to be approved with Backend/Billing.

### 19.4 AR-013 authoritative writer — comparison against live documents

| Bounded record/decision | Writer A (recommended PROPOSED) | Writer B (OPEN alternative) | Current source conflict |
|---|---|---|---|
| IdentityCredentials, MFA, sessions, global principal | Identity/Auth (Security identity subdomain) | same | no meaningful conflict |
| Tenant/Company/Branch graph | Multi-Tenant | same | organization/domain ownership aligned |
| TenantMembership state/version | **Multi-Tenant single writer** | Security/IAM single writer; Multi-Tenant gets read-only projection | `MODULE_REGISTRY` says Security membership; Security §17, Backend §17 and MT §16 favor Multi-Tenant; Billing §2/ADR-BILL-010 says IAM membership |
| RoleAssignment, Grant and policy | Security/Permissions single writer | same | Security §17 aligned |
| PermissionEvaluation | Security/Permissions owns versioned policy/decision, Backend/domain guards enforce with authoritative resource ownership | same | aligned; RLS does not grant company/branch permission |
| Seat charge eligibility | Billing | same | membership state does not imply paid Seat |

**Recommendation:** choose Writer A after cross-domain approval because organization membership belongs naturally with tenant graph and isolation, while credentials remain global and grants belong to Security. Writer B is viable if IAM owns membership in the final domain map; Multi-Tenant must then never update it. **No dual writable tables or bidirectional status overwrites.** A single membership ID and monotonic state/version for event dedup.

**Lifecycle PROPOSED:** invited (no access), active, suspended (deny), revoked (deny); expiry/declined and reactivation vs new ID OPEN. Invite acceptance verifies identity and tenant. On tenant switch, re-fetch active membership, policy version and scoped grants; server rebinds context and invalidates previous tenant cache/subscription context. On revoke/grant removal, monotonic change version + outbox notification invalidates snapshots, but **notification/cache invalidation alone never authorizes**: all critical operations, delayed job, offline sync and long-running mutation recheck authoritative membership/permission before side effect; unavailable authority ⇒ deny/retry, not allow. Backend guards and domain ownership validation remain mandatory.

### 19.5 MR-11 negative-test acceptance matrix (DESIGN ONLY — NOT EXECUTED)

| ID | Scenario / attack | Required result (PROPOSED acceptance) | Primary joint reviewer |
|---|---|---|---|
| MT-NEG-01 | Tenant A token with Tenant B header/resource ID; list, detail, write, count, export | deny/no B metadata; no B mutation | Security/Backend |
| MT-NEG-02 | Active Tenant A→B switch, stale A token, cached A query or open subscription | B context newly verified; no A privileged action/data in B | Security/UX |
| MT-NEG-03 | Revoked or suspended membership, cached role and stolen refresh, queued offline action | deny/re-auth; no committed side effect, audit | Identity/Security/Mobile |
| MT-NEG-04 | RoleAssignment revoked, old policy_version/scope token | current decision deny even if event notification delayed | Security/Backend |
| MT-NEG-05 | Branch A user accesses Branch B in same company or foreign company, guessed document ID | deny unless explicit authorized grant; tenant-only RLS not enough | Security/ERP domains |
| MT-NEG-06 | Company A employee attempts writing company B records, mismatched branch.company_id | reject ownership mismatch and no cross-company write | Multi-Tenant/Database |
| MT-NEG-07 | Background job missing tenant context or reusing stale membership | fail closed; worker cannot touch wrong tenant | Backend/DB |
| MT-NEG-08 | Cache key/cached search facet, autocomplete, recent, index rebuild or global count lacks tenant/scope | no cross-tenant or unauthorized branch inference | Backend/Security/Search |
| MT-NEG-09 | Same pooled DB connection used for tenant A then B, rollback/retry/prepared statement/cursor | transaction-local context isolated; missing context returns no rows/errors | Database/Security |
| MT-NEG-10 | Runtime role superuser/BYPASSRLS/table owner/SECURITY DEFINER/view alternate route | runtime cannot bypass; privileged paths audited separately | Database/Security |
| MT-NEG-11 | Stale placement_epoch, move pooled→dedicated, simultaneous old writer/queue retry | old writer fenced, no double write/charge, deterministic recovery | Infra/DB/Billing |
| MT-NEG-12 | Tenant-specific backup/restore or export imported into wrong tenant/shard | restore must preserve identifiers and avoid foreign tenant exposure | Infra/DB |
| MT-NEG-13 | Client submits foreign billing_account_id/wallet_id or branch reassigned after fact | ignored/rejected; Billing resolves effective payer, historical attribution preserved | Billing/Security |
| MT-NEG-14 | Company/branch scope removed mid-flight before business commit | recheck or transactional guard deny; no unauthorized side effect | Backend/Security |

Acceptance requires reproducible fixtures for ≥2 tenants, ≥2 companies/branches, overlapping global user memberships, revoked roles, pooled connections, retries and placement epochs. For each case check **read + mutation + side-channel** and audit evidence. These are planned tests only, **NOT TESTED / NOT VERIFIED**; automated gating before production remains PROPOSED. No numerical performance/SLO/restore guarantee invented.

### 19.6 Placement / sharding evolution remains PROPOSED

Stage P (candidate): pooled shared schema + composite tenant FK and defense-in-depth PostgreSQL RLS with non-bypass runtime role and transaction-local verified context, if PostgreSQL selected. Stage E: contract/compliance/performance-gated dedicated DB/instance. TenantRegistry `tenant_id → placement_id + placement_epoch`; move uses single-writer fencing, snapshot/checksum/ledger reconciliation, verified cutover, rollback plan. RLS does not replace Security permission checks; dedicated DB does not replace membership authorization. Per-schema is an alternative with migration overhead. Later pooled tenant-based shard reassignment (not Company/Branch hash sharding). Region/residency, thresholds, RPO/RTO, cutover downtime, CDC and per-tenant restore remain OPEN; DB/Security/Infrastructure sign-off required.

### 19.7 Explicit unresolved approvals / contradictions

- **AR-001/MR-01:** legal debtor and payer Party, company/legal-entity and account/wallet cardinality, cross-company consolidated invoice, third-party topup, split seat/resource allocation and effective-dated account changes — **OPEN: user + Billing + Finance + Master**.
- **AR-013:** authoritative TenantMembership writer remains **CONFLICTING**: global registry and Billing legacy wording vs Security/Backend/Multi-Tenant candidates. Resolve with explicit one-writer ADR decision, do not silently modify registry.
- **MR-11:** PostgreSQL/shared-schema RLS final choice, runtime/maintenance roles, privileged break-glass, placement/restore/migration, isolation tests — **PROPOSED / OPEN**.
- **AR-009/010 dependent:** offline event's billable calendar month and document lifecycle charge trigger are still OPEN; not solved by tenant context or migration.
- No central registry updates in this Round 2 patch; no old approved business rule is changed.

See [ADR-MT-001](../adr/ADR-MT-001.md) and [ADR-MT-002](../adr/ADR-MT-002.md) for decision alternatives and risks.


## 20. Round 2 live-source alignment delta (2026-10-12; PROPOSED, OPEN)

This is an incremental reconciliation against **Billing §20**, **Backend §18**, **Database §18**, **Security §17–19**, and **MODULE_REGISTRY Round 2 v0.2**, read on current `main`. It updates source crosswalk only; previous §19 cardinality comparison and MT-NEG-01..14 remain the canonical proposed matrix. Nothing below is a new APPROVED choice or a claim of executed tests.

### 20.1 Current owner proposals versus still-open contradictions

| Fact / operation | Current candidate writer | Reader / enforcer | Unresolved contradiction |
|---|---|---|---|
| Global credentials, MFA, session/device validity | Identity/Auth (Security identity domain) | Backend authenticates; Multi-Tenant references user_id only | Authentication package and device/session policy OPEN |
| Tenant/Company/Branch affiliation | Multi-Tenant | Billing source attribution, Security scope, Backend guard | Company↔LegalEntity mapping authority and change history shared with Finance OPEN |
| TenantMembership canonical state/version | **Multi-Tenant recommended candidate** | Security evaluation, Billing seat input, Backend authoritative recheck | MODULE_REGISTRY original Security membership umbrella and Billing old IAM wording still exist; latest registry v0.2 explicitly reports conflict; no single writer APPROVED |
| RoleAssignment/grants and PermissionEvaluation | Security/Permissions candidate writer/evaluator | Backend/domain owns operation enforcement and object relationship | Registry Round 2 mentions alternative Multi-Tenant grant owner or split administrative API; a split UI/API does not justify dual mutable grant owners |
| SaaS BillingAccount, Wallet, payer/debtor liability ledger | Billing | Multi-Tenant validates source organization; Finance maps LegalEntity | Wallet/account cardinality, legal debtor, paying actor, payer authorization and consolidation all OPEN |

Latest **Billing §20.1** distinguishes `payer_party_id` (funding actor) from `legal_debtor_entity_id` (liability), with `payer_authorization_id?` and immutable `mapping_version`. Do not conflate either with tenant_id or assume branch billing actor = legal debtor. Earlier Billing §3 one-wallet-per-company wording is superseded in interpretation by Billing §19/§20 **as a PROPOSED candidate only**, not a new approved status. The latest Round 2 registry **documents** (does not resolve) the earlier Security-membership conflict. The old primary module §15/§19 statements continue to be proposals.

### 20.2 Integration clarifications to Billing §20 / Backend §18 (PROPOSED)

- Source organization validation: immutable `tenant_id`; `source_company_id?`, `source_branch_id?`, checked against company/branch affiliation at effective time; `membership_id?` and `membership_version?` tie actor to the real authorized context. For global Seat or shared Resource Usage, company/branch may be null instead of guessed.
- Separate timestamps: source `occurred_at`, server `committed_at`, ingestion `received_at`; Billing alone chooses charge-effective month (AR-009/MR-02 OPEN). If backdated or reassigned company, preserve effective-dated source and Billing payer mapping snapshots. Multi-Tenant does not choose calendar-month grace policy.
- Billing resolves `billing_account_id`, `wallet_id`, `payer_party_id?`, `legal_debtor_entity_id?`, `payer_authorization_id?`, `mapping_version` — never accept untrusted submitted wallet IDs as routing authority. Ambiguous mapping → pending/rejected (per Billing policy), not implicit pooled balance.
- Backend §18 distinguishes **event_id** (delivery envelope) from **business_occurrence_id** (same semantic operation). Earlier §15 `UsageFactRecorded.v1` must be supplemented with `business_occurrence_id` and `aggregate_revision?`; replay with a fresh event_id but identical business_occurrence_id cannot generate a second economic debit. Producer emits facts from committed owner transaction/outbox; Billing owns pricing, rating, posting and deduplication.
- Membership event proposed `MembershipChanged.v1`: tenant_id, membership_id, user_id, old/new state, monotonic membership_version, effective_at, committed_at, transition_id, changed company/branch scopes if valid. Billing determines Seat eligibility and prorating; a revoked permission need not automatically change commercial license billing unless separately approved.

**Additional acceptance refinement:** extend existing MT-NEG-13 with payer_party ≠ legal_debtor, third-party funder authorization, effective-dated company transfer and split-accrual replay; extend MT-NEG-11 with two event IDs sharing one business_occurrence_id across placement cutover, expecting one Billing economic posting. These are **test designs not run**. Billing §§20.2–20.5 calendar-month, seat status, document trigger and closed-month treatment remain OPEN.

**Approval gate:** user + Billing + Finance confirm payer/debtor/Wallet cardinality and shared-seat allocation; Security + Multi-Tenant + Backend + Master confirm single writer and revocation boundary; Database + Security + Infrastructure approve selected shared-schema/RLS or dedicated placement/restore strategy. No implementation authorization or changes to central registries.

---

## 21. AR-001 / MR-01 — User-approved MVP billing cardinality (2026-10-12)

**APPROVED_BY_USER:** Tenant may contain multiple Companies; each MVP Company has its own distinct LegalEntity, BillingAccount and Wallet (1:1:1:1); Branch no wallet. Company LegalEntity bears its SaaS debt; authorized third-party top-up does not move debt. Seat same user within one Company is not multiplied by branches/devices, but separately counted per distinct Company. Tenant-shared measured Resource Usage is billed to one preassigned responsible Company; Company/Branch usage to owning Company. [ADR-BILL-011](../adr/ADR-BILL-011.md). Old §15/19/20 option tables are comparative history; do not interpret former OPEN MVP cardinality wording as current approval status. Enterprise consolidated payer stays OPEN.

**PROPOSED:** Multi-Tenant validates organization/legal linkage/effective membership, Billing owns account/wallet/rating and assignment snapshot; Security checks role/scope. Tenant admin proposes and receiving Company's financial representative accepts responsible-company switch; assignment future-effective, historical debtor/credits unchanged and no guessed wallet, automatic offset or unauthorized retroactive charge. Exact approval delegation, metering cutoff/timezone, LegalEntity merge/migration and compliance remain OPEN.

---

## 22. Seat source-of-truth handoff to Billing — ADR-BILL-012 (2026-10-12)

Business **APPROVED_BY_USER** via [ADR-BILL-011](../adr/ADR-BILL-011.md): one Seat per canonical user per Company despite multiple Branches/devices, distinct Seat scope for user in another Company. [ADR-BILL-012](../adr/ADR-BILL-012.md) refines the **PROPOSED** use of effective-dated `TenantMembership` status/version/company-scopes from one MT writer (AR-013 owner sign-off remains OPEN), Security grants/revocation and Billing-owned rated `user_id × company_id` intervals. Joining/suspending/transfer must emit distinct effective facts and not issue financial commands or client wallet IDs. Same user in two branches cannot be multiplied by source membership row count. Day-cutoff, charging status, price, rounding, late correction and monthly Seat cycle remain OPEN.

---

## 23. Delegated architecture handoff — membership, payer mapping and company attribution (2026-10-12)

[ADR-010](../adr/ADR-010-mvp-delegated-baseline.md) fixes a preferred **PROPOSED** one-writer map for handoff: Multi-Tenant owns Tenant/Company/Branch and TenantMembership lifecycle/status/verified company scope; Security owns grants and authorization, Billing owns priced economic Seat snapshots, identity principal belongs Security Auth component. **APPROVED_BY_USER** MVP Company distinct LegalEntity/BillingAccount/Wallet and per Company Seat + designated Tenant-shared resource (ADR-BILL-011) remain binding. No cross-company amount pooling, automatic legal debt transfer or guessed Wallet. Distinct source attribution, active financial assignment, legal debtor and actual funder must be persisted. Reassignments prospective with versioned provenance; Billing owns financial assignment and independent ledger. Technical RLS/pooling migration tests not run; membership final ADR sign-off pending. [Handoff](../handoff/CODEX_START_HERE.md).
