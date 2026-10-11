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
