# ADR-SEC-006 — Impersonation

**Status: PROPOSED** · 2026-10-11

## Muammo
Multi-tenant ERP SaaS doirasida Impersonation xavfsizlik modelini belgilash.

## Variantlar
Unrestricted operator access; no impersonation.

## Tavsiya etilgan yechim (tasdiqlanmagan)
Support access time boxed, approved or audited break glass, MFA, original actor+acting-as, sensitive action restricted.

## Sabab va xavflar
Insider risk, approval overhead.

## Boshqa modullarga ta’sir
UI, Support, Audit.

## Qaror statusi
Ushbu yechim hali APPROVED emas. Tegishli modullar bilan muvofiqlashtirish va reviewdan keyin tasdiqlash mumkin. Kod yozilmaydi.

[Security Architecture](../modules/SECURITY_ARCHITECTURE.md)

## Round 2 cross-module contract (2026-10-12) — PROPOSED / OPEN
See [Security Architecture §21](../modules/SECURITY_ARCHITECTURE.md#21-round-2-correction--ar-013-p0--mr-11--ar-006-2026-10-12) for the authoritative single-writer matrix, alternative membership ownership A/B, source conflict register, revoke/tenant-switch cache lifecycle, mobile stolen-token/device revoke/impersonation rules, SEC-R2-01…13 negative-test acceptance matrix and required evidence. This ADR remains **PROPOSED**; AR-013 and MR-11 remain **OPEN**; no test or benchmark has been run.
