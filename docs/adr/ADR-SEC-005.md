# ADR-SEC-005 — Audit trail

- **Status:** PROPOSED
- **Date:** 2026-10-11
- **Owner:** Security Architecture
- **Source:** [Security Architecture](../modules/SECURITY_ARCHITECTURE.md)

## Muammo
Ko‘p tenantli ERP SaaS uchun «Audit trail» bo‘yicha izchil, audit qilinadigan, xavfsiz model tanlash kerak.

## Variantlar
Mutable application logs; logging only successes.

## Tavsiya etilgan yechim (hali tasdiqlanmagan)
Critical event append-only, tamper-evident audit with actor/acting-as, tenant, action, resource, time, outcome.

## Sabab va xavflar
Forensics va accountability; PII exposure, retention/storage costs.

## Modullarga ta’siri
Finance, Billing, Admin, Security operations.

## Qaror sharti
Security, tegishli modul va Master Architecture muvofiqlashtirilgandan keyingina APPROVED bo‘lishi mumkin. Hozircha hech qanday kod yoki migratsiya yozilmaydi.
