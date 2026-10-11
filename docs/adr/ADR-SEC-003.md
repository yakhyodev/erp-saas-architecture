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
