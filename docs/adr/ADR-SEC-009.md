# ADR-SEC-009 — API security

- **Status:** PROPOSED
- **Date:** 2026-10-11
- **Owner:** Security Architecture
- **Source:** [Security Architecture](../modules/SECURITY_ARCHITECTURE.md)

## Muammo
Ko‘p tenantli ERP SaaS uchun «API security» bo‘yicha izchil, audit qilinadigan, xavfsiz model tanlash kerak.

## Variantlar
Gateway-only policy; endpoint authentication without resource checks.

## Tavsiya etilgan yechim (hali tasdiqlanmagan)
Object-level auth, validation, scoped throttle, replay/idempotency defense, secure files/webhooks and CSRF/CORS controls.

## Sabab va xavflar
Limits IDOR and abuse; operational false positives.

## Modullarga ta’siri
Backend, Integration, Mobile, Billing.

## Qaror sharti
Security, tegishli modul va Master Architecture muvofiqlashtirilgandan keyingina APPROVED bo‘lishi mumkin. Hozircha hech qanday kod yoki migratsiya yozilmaydi.

## AR-013/AR-006 server-guard contract — PROPOSED
All HTTP, sync, job, file and export operations resolve active identity/session/device, active tenant membership/version, branch graph and grants/policy version; authorize current resource/action server-side immediately before side effects. Client-provided tenant ID is a requested selector, not authority. Switching tenant cannot carry permissions from prior tenant. Offline command operation_id idempotent and original device time untrusted; revoked devices/memberships reject rather than post charges/stock/GL. Timeout unknown; retry same operation_id. Device auth strategy/MFA/TTL and mobile response taxonomy joint Security/Mobile/Backend OPEN.

## Round 2 cross-module contract (2026-10-12) — PROPOSED / OPEN
See [Security Architecture §21](../modules/SECURITY_ARCHITECTURE.md#21-round-2-correction--ar-013-p0--mr-11--ar-006-2026-10-12) for the authoritative single-writer matrix, alternative membership ownership A/B, source conflict register, revoke/tenant-switch cache lifecycle, mobile stolen-token/device revoke/impersonation rules, SEC-R2-01…13 negative-test acceptance matrix and required evidence. This ADR remains **PROPOSED**; AR-013 and MR-11 remain **OPEN**; no test or benchmark has been run.
