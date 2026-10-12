# ADR-SEC-002 — Authorization RBAC/ABAC

**Status: PROPOSED** · 2026-10-11

## Muammo
Multi-tenant ERP SaaS doirasida Authorization RBAC/ABAC xavfsizlik modelini belgilash.

## Variantlar
RBAC alone; client-side role checks.

## Tavsiya etilgan yechim (tasdiqlanmagan)
Server-enforced deny-by-default RBAC plus tenant/branch/object/status ABAC, separation of duties, no self escalation.

## Sabab va xavflar
Policy complexity; authorization cache staleness.

## Boshqa modullarga ta’sir
All modules.

## Qaror statusi
Ushbu yechim hali APPROVED emas. Tegishli modullar bilan muvofiqlashtirish va reviewdan keyin tasdiqlash mumkin. Kod yozilmaydi.

[Security Architecture](../modules/SECURITY_ARCHITECTURE.md)

## AR-013 authoritative boundary addendum — PROPOSED
Candidate owner: Security/Permissions writes tenant-scoped RoleAssignment, grant/delegation and policy versions; Multi-Tenant writes membership states/versions and org graph; Identity owns global principal/session. Security.PermissionEvaluation is canonical shared policy decision; Backend and each domain enforce server-side object/action guards, reading authoritative scope facts. Proposed evaluate inputs: principal+session/device, tenant_id, active membership_id/version, assignment_version, policy_version, company/branch, authoritative object state, action, MFA/step-up and clock. Default deny on missing/mismatch/stale states. Tenant switch reissues/rebinds verified active context; old context invalid as authority for new tenant. Role or membership revocation emits outbox invalidation hints, but each critical mutation/sync must re-check source state. No client claims as standalone authority. AR-013 remains OPEN for joint decision and rejoin, delegation, service identities, caching TTL.

## Round 2 cross-module contract (2026-10-12) — PROPOSED / OPEN
See [Security Architecture §21](../modules/SECURITY_ARCHITECTURE.md#21-round-2-correction--ar-013-p0--mr-11--ar-006-2026-10-12) for the authoritative single-writer matrix, alternative membership ownership A/B, source conflict register, revoke/tenant-switch cache lifecycle, mobile stolen-token/device revoke/impersonation rules, SEC-R2-01…13 negative-test acceptance matrix and required evidence. This ADR remains **PROPOSED**; AR-013 and MR-11 remain **OPEN**; no test or benchmark has been run.
