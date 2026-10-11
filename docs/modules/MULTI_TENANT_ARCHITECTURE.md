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
