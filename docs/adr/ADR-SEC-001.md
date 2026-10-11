# ADR-SEC-001 — Authentication

**Status: PROPOSED** · 2026-10-11

## Muammo
Multi-tenant ERP SaaS doirasida Authentication xavfsizlik modelini belgilash.

## Variantlar
Unrevocable JWT sessions; identity per tenant.

## Tavsiya etilgan yechim (tasdiqlanmagan)
Central identity and active tenant membership, MFA for privileged users, revocable server-side sessions, WebAuthn preference, mobile secure storage.

## Sabab va xavflar
MFA friction, offline revocation delay.

## Boshqa modullarga ta’sir
Backend, UI, Mobile.

## Qaror statusi
Ushbu yechim hali APPROVED emas. Tegishli modullar bilan muvofiqlashtirish va reviewdan keyin tasdiqlash mumkin. Kod yozilmaydi.

[Security Architecture](../modules/SECURITY_ARCHITECTURE.md)
