# ADR-SEC-007 — Encryption

**Status: PROPOSED** · 2026-10-11

## Muammo
Multi-tenant ERP SaaS doirasida Encryption xavfsizlik modelini belgilash.

## Variantlar
Only transport encryption; bespoke homegrown crypto.

## Tavsiya etilgan yechim (tasdiqlanmagan)
TLS, encryption at rest and backups, KMS envelope for sensitive fields, rotation/audit, encrypted offline device data.

## Sabab va xavflar
Key availability and rotation risk.

## Boshqa modullarga ta’sir
Database, Infrastructure, Mobile.

## Qaror statusi
Ushbu yechim hali APPROVED emas. Tegishli modullar bilan muvofiqlashtirish va reviewdan keyin tasdiqlash mumkin. Kod yozilmaydi.

[Security Architecture](../modules/SECURITY_ARCHITECTURE.md)
