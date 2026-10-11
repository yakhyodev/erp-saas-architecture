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

## MR-11/AR-006/AR-013 negative acceptance matrix — PROPOSED, NOT EXECUTED
Test categories: tenant A cannot read/write tenant B detail/list/count/search/autocomplete/export/files, branch scope denial, forged tenant selector, stale membership/grant/role/policy versions, revoked session/device, concurrent tenant switch, queued offline replay, timeout idempotency. If Postgres/RLS selected: same pooled connection A then B with transaction context and rollback; missing context; runtime role bypass denied; prepared statement/job/worker/batch isolation; privileged maintenance paths reviewed; stale tenant placement. Also verify cache/search/storage/CDN cross-scope and audit event integrity. Automated release gate proposed; no passing tests, production isolation or security certification asserted.
