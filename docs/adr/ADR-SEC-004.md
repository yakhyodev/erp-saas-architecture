# ADR-SEC-004 — PostgreSQL RLS

**Status: PROPOSED** · 2026-10-11

## Muammo
Multi-tenant ERP SaaS doirasida PostgreSQL RLS xavfsizlik modelini belgilash.

## Variantlar
Application filter alone; RLS-only permission model.

## Tavsiya etilgan yechim (tasdiqlanmagan)
If PostgreSQL approved use ENABLE FORCE RLS with non-bypass app role, transaction local trusted tenant context, pool reset and privileged account audit.

## Sabab va xavflar
Connection reuse context leakage; privileged function bypass.

## Boshqa modullarga ta’sir
Database, Backend, Infrastructure.

## Qaror statusi
Ushbu yechim hali APPROVED emas. Tegishli modullar bilan muvofiqlashtirish va reviewdan keyin tasdiqlash mumkin. Kod yozilmaydi.

[Security Architecture](../modules/SECURITY_ARCHITECTURE.md)

## MR-11 RLS/pooling contract addendum — PROPOSED, UNTESTED
Only if PostgreSQL/shared schema/RLS chosen jointly: tenant-owned row RLS predicates for read and WITH CHECK write; runtime app role non-superuser, no BYPASSRLS/table owner authority; transaction-local tenant context with fail-closed default; pool transaction reuse/rollback/retry, prepared statements and worker batches must not inherit former tenant context. Privileged maintenance, SECURITY DEFINER/views, backup/restore and tenant placement migration require separate audited paths; ordinary app cannot bypass RLS. Candidate negative tests: A→B reuse same connection, missing context denial, forged tenant header, cross-tenant read/write/list, privileged runtime bypass attempts, stale placement_epoch, restore/cutover; all **NOT RUN**. DB owns physical schema/pool configuration, Multi-Tenant placement, Security policy/acceptance. MR-11 OPEN.
