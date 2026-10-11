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
