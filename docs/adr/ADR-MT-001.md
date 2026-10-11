# ADR-MT-001 — Tenant isolation va placement strategiyasi

**Status:** PROPOSED  
**Date:** 2026-10-11  
**Owner:** Multi-Tenancy Architecture  
**Approval dependencies:** Master Architecture, Database, Security, Backend, Billing, Infrastructure.

## Muammo
Minglab tenantlar yagona ERP SaaSda arzon resurs bilan ishlashi, qat’iy data isolation va yirik enterprise uchun kuchliroq ajratish hamda scaling imkoniyati bo‘lishi lozim.

## Variantlar
1. Shared DB/shared schema, faqat application filter — arzon, ammo data leak xavfi baland; **rad etish taklifi**.
2. Shared DB/shared schema + PostgreSQL RLS — arzon, markazlashgan migrations; RLS/context discipline zarur.
3. Schema per tenant — logical isolation, lekin minglab schema va operatsion migrations qimmat.
4. Database per tenant — kuchli isolation, lekin kichik tenantlar uchun resurs xarajati katta.
5. Hybrid pooled DB + dedicated enterprise DB + tenant registry — moslashuvchan, biroq routing/migration murakkab.

## Tanlangan yechim (PROPOSED)
Boshlanishida **2-variant**, evolyutsion arxitektura sifatida **5-variant**. Stable tenant_id, har tenant-owned table uchun tenant_id va RLS; server-verified tenant context; placement registry; keyinchalik pooled shardlar yoki dedicated DBga ko‘chirish.

## Sabablar
Boshlang‘ich sarfni kamaytirish; DB qatlamida qo‘shimcha isolation; katta mijozlarga differensial deployment; shard rebalancingga tayyorlik.

## Xavflar va kamaytirish
- Data leak → RLS, default deny, non-bypass runtime DB role, negative tests va audit.
- Noisy neighbor → tenant-aware limits, timeouts, metrics, dedicated move.
- Split-brain migration → fencing, placement_epoch, verified cutover.
- Privileged bypass → admin/migration rolesni ajratish, threat model.
- Duplicate usage billing → stable IDs va idempotency.
- Tenant-specific recovery → per-tenant backup/export/restore drills (RPO/RTO OPEN).

## Boshqa modullarga ta’siri
Backend/Master: context + routing abstraction. Database: tenant keys/RLS/pool safety. Security: membership grants/revocation. Billing: usage attribution (billing_account mapping OPEN). Infrastructure: registry HA, dedicated placement va restore. Storage/Search/Cache: namespace isolation.

## Tasdiqlash shartlari
Master, Database, Security, Backend, Billing va Infrastructure reviewlari; billing ownership va migration threat model masalalari hal bo‘lishi. Ungacha **PROPOSED**.


## 2026-10-11 reconciliation clarification (PROPOSED, MR-11)

This ADR does **not** approve PostgreSQL, shared-schema/RLS, placement, dedicated DB, shard split, tenant-level restore or privileged bypass. They remain comparative options subject to Database/Security/Infra/Master validation.

**Hard isolation acceptance target (PROPOSED design invariant):** globally unique stable tenant_id; tenant-owned rows and object/search/cache/job envelopes are tenant-scoped; no cross-tenant read/write/metadata leakage via user requests or background processing; client-selected tenant/company IDs do not confer access; effective member + scoped authorization + resource ownership required; stale/unknown context fails closed.

**RLS proposal:** app runtime role cannot BYPASSRLS/superuser/table-owner-bypass; both read and write policies; transaction-local verified context only; reject absent/mismatched context; FORCE RLS and owner-role threat model; negative tests for connection pool reuse, SECURITY DEFINER/functions, views, batch jobs, imports, reports, cache, search and exports. Explicit operational migration role segregated and audited. Privileged break-glass is OPEN: default disabled, JIT/ticket, expiry, narrow-scoped traceable exception; never silently run customer requests as privileged DB user.

**Tenant placement/migration candidate:** TenantRegistry maps immutable tenant_id to placement_id + epoch; epoch fences old writers. Pooled → dedicated migration: snapshot, consistency verification, replay decision (CDC/dual-write neither approved), single-writer cutover, idempotent event processing, rollback plan. Tenant-scoped backup/restore, RPO/RTO, legal retention, data residency and no-downtime promise OPEN.

**Compatibility with AR-001/MR-01:** DB isolation tenant does not imply Tenant = Company = LegalEntity = BillingAccount = Wallet. These relational choices and debt owner stay OPEN; Billing owns wallet ledger/charges, Finance legal entity mapping. Company/branch scope needs additional server authorization beyond RLS.

**Decision gate:** shared-schema/RLS final choice; DB ownership privileges; tenant movement and per-tenant restore proven via threat model/tests and signed off by Database, Security, Infra, Backend, Billing, Master. Until then **PROPOSED; MR-11 OPEN**.
