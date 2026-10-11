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
