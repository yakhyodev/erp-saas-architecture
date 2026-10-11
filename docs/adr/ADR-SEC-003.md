# ADR-SEC-003 — Tenant isolation

- **Status:** PROPOSED
- **Date:** 2026-10-11
- **Owner:** Security Architecture
- **Source:** [Security Architecture](../modules/SECURITY_ARCHITECTURE.md)

## Muammo
Ko‘p tenantli ERP SaaS uchun «Tenant isolation» bo‘yicha izchil, audit qilinadigan, xavfsiz model tanlash kerak.

## Variantlar
Faqat API filters; schema-per-tenant; database-per-tenant.

## Tavsiya etilgan yechim (hali tasdiqlanmagan)
Trusted tenant context, tenant-owned keys va composite relationships; app enforcement + DB defense in depth.

## Sabab va xavflar
Cross-tenant leakning oldi olinadi; shared DB scale va accidental bypass xavfi.

## Modullarga ta’siri
Multi-Tenant, Database, Files, Billing, Analytics.

## Qaror sharti
Security, tegishli modul va Master Architecture muvofiqlashtirilgandan keyingina APPROVED bo‘lishi mumkin. Hozircha hech qanday kod yoki migratsiya yozilmaydi.

## AR-013/MR-11 isolation alignment — PROPOSED
Tenant/Company/Branch hierarchy and membership lifecycle candidate source is Multi-Tenant; Security consumes scope references, never creates tenancy graph. All guards derive tenant from verified membership; original session+tenant switch+grant version bound context. Tenant-owned queues, caches, attachments, search, exports, reports and billing events carry scope. Application checks precede DB defense in depth. Shared schema/dedicated/sharding decisions stay OPEN; no redefinition of BillingAccount/Wallet cardinality. Test scenarios are design only, not executed.
