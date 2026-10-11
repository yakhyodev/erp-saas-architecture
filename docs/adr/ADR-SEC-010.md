# ADR-SEC-010 — Security verification

- **Status:** PROPOSED
- **Date:** 2026-10-11
- **Owner:** Security Architecture
- **Source:** [Security Architecture](../modules/SECURITY_ARCHITECTURE.md)

## Muammo
Ko‘p tenantli ERP SaaS uchun «Security verification» bo‘yicha izchil, audit qilinadigan, xavfsiz model tanlash kerak.

## Variantlar
Only happy-path unit tests; manual pentest alone.

## Tavsiya etilgan yechim (hali tasdiqlanmagan)
Cross-tenant/branch denial matrix, RLS pool/job/cache/file tests, session revocation and audit integrity as release gates.

## Sabab va xavflar
Regression resistance; test fixture and execution costs.

## Modullarga ta’siri
QA, Database, All modules.

## Qaror sharti
Security, tegishli modul va Master Architecture muvofiqlashtirilgandan keyingina APPROVED bo‘lishi mumkin. Hozircha hech qanday kod yoki migratsiya yozilmaydi.
