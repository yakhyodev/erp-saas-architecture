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
