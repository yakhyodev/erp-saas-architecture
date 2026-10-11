# ADR-BILL-003 — Usage event idempotency

- **Status:** PROPOSED
- **Date:** 2026-10-11
- **Scope:** Billing Architecture
- **Business-rule authority:** APPROVED biznes invariantlari [Billing module](../modules/BILLING_ARCHITECTURE.md) da; bu ADR texnik usulni tasdiqlamaydi.

## Muammo
At-least-once delivery, offline replay va retried event ikki marta haq chiqarishi mumkin.

## Ko'rilgan variantlar
Synchronous billing call; exactly-once transportga ishonish; inbox/outbox + unique economic posting.

## Tanlangan yechim (taklif)
PROPOSED: transactional outbox/inbox, tenant+producer+source event+charge-type dedup, unique constraints, durable retry.

## Sabab
Tarmoq va worker xatosida iqtisodiy exactly-once effect.

## Xavflar va ochiq masalalar
Source ID o'zgarishi, document amendment va out-of-order event; compensation policy OPEN.

## Boshqa modullarga ta'siri
Sales, Inventory, Mobile, Backend, Database.

## Qaror tartibi
Foydalanuvchi alohida tasdiqlamaguncha PROPOSED. O'zgartirilsa eski ADR SUPERSEDED sifatida tarixda qoladi.
