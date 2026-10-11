# ADR-SEC-008 — Secrets management

- **Status:** PROPOSED
- **Date:** 2026-10-11
- **Owner:** Security Architecture
- **Source:** [Security Architecture](../modules/SECURITY_ARCHITECTURE.md)

## Muammo
Ko‘p tenantli ERP SaaS uchun «Secrets management» bo‘yicha izchil, audit qilinadigan, xavfsiz model tanlash kerak.

## Variantlar
Static secrets in repo/environment-only without lifecycle.

## Tavsiya etilgan yechim (hali tasdiqlanmagan)
Central secret manager, short-lived identities, per-environment and tenant-scoped integration credentials, rotation.

## Sabab va xavflar
Reduced credential exposure; vault availability and rotation compatibility.

## Modullarga ta’siri
Infra, Integration, Backend.

## Qaror sharti
Security, tegishli modul va Master Architecture muvofiqlashtirilgandan keyingina APPROVED bo‘lishi mumkin. Hozircha hech qanday kod yoki migratsiya yozilmaydi.
