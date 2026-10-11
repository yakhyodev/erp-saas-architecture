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
