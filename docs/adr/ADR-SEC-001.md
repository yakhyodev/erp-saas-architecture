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

## AR-013/AR-006 alignment addendum — PROPOSED
Identity/Auth is the candidate single writable owner of global user, UserCredentials, MFA, sessions, device registration/security status and credential/session revocation. Multi-Tenant owns TenantMembership state (candidate, AR-013 OPEN); authentication never confers tenant authorization by itself. Server checks device/session validity at each online/mobile sync acceptance, short-lived credentials/refresh rotation and revocation/version checks. Expiry, SSO, MFA scope, device attestation and offline TTL remain OPEN. Device offline remote wipe is not guaranteed. Backend implements guards but does not gain duplicate credential authority. No implementation or tests performed.
